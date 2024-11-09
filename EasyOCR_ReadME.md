MapleStory Mobile (MSM) is a mobile MMORPG inspired by the popular MMORPG, Maplestory, by Nexon. In essence, it's a game about gathering resources to enhance equipment and conquer challenging endgame content. To maximize resource accumulation, players often seek to optimize their gameplay. The introduction of the Auto-Battle (AB) system in MSM has provided a new opportunity for data-driven decision-making.

The EasyOCR library is well-suited for this task due to its simplicity. Our use case involves scanning screenshots of AB sessions to extract relevant statistics for future analysis. Given the consistent nature of the screenshots, we anticipate minimal variation in extraction results.

<img src="https://github.com/user-attachments/assets/b74899d5-71f0-4862-ba1d-935080a2a641" width="250" height="200"/>

Figure 1.0 - Example screenshot of a Auto-Battle session

*Note - The important statistics include _duration, kill count, meso gained, red meso gained and total exp gained_.

The EasyOCR library scans through each screenshot to extract the important statistics. Extra  measures are taken to validate and rectify erroneous extraction results. These results are then stored into dataframe for further data processing. 

<img src="https://github.com/user-attachments/assets/080ca1ed-b0fa-4924-afe8-595289557182" width="250" height="200"/>

Figure 1.1 - Extracted statistic (highlighted in green boxes) from Auto-Battle session

# Python Libraries

<details>
  <summary>The EasyOCR Library</summary>
  
The EasyOCR library is used to extract text from each screenshot. A _Reader_ object is first intialised into the variable '_reader_'. After which the image is parse into the _.readtext()_ function and the result is returned.  
  ```
  reader = easyocr.Reader(['en'], gpu = False)
  result = reader.readtext(image_path)
  ```
</details>

<details>
  <summary>The CV2 Library</summary>
  
The CV2 library is used to visualise each extracted element of the image through the _.imread()_ function. Refer to Figure 1.1 for an example
  ```
  # Visualise each extracted element of the image using the function - checkImage
  img = cv2.imread(image_path,1)
  checked_img = checkImage(img, result)
  plt.imshow(checked_img)
  plt.show()
  
  # Save the image to the desktop
  plt.imsave('sample_image.jpg', checked_img)
  ```
</details>


<details>
  <summary>The glob Library</summary>
  
The glob library is used to perform pattern matching for files and directories. It provides a simple way to find files that matches a specific pattern.
  ```
list_of_image =  glob.glob(r'ab_*.jpg')
  ```

Using the above line of code, filenames starting with the prefix "ab_" are selected as Auto-Battle screenshots have a prefix of "ab_"
</details>



<details>
  <summary>Things to note</summary>
  <img src="https://github.com/user-attachments/assets/141135a9-f876-4aa9-b6d4-99ce034401dc">

  - Notice that the total exp figure extract is off by one digit. This has to be manually amended. One potential way to catch this error is through outlier dectection.

  ```
  duration = inner_list[0][1].replace(inner_list[0][1][-3],':')
  ```
  - In some cases, we observe that the duration statistic is invalid as the colon has been misread as a period by the EasyOCR library. To ensure the accuracy our result, the above line of code replaces the any punctuation found in the duration statistic with a colon. At the moment, these mistakes have to be manually fixed until a better solution is implemented.

</details>

<details>
  <summary>Statistical Functions (Scipy.stat)</summary>

  - Perform various parametric or non-parametric statistical tests (e.g. ANOVA, t-Test, kurskal-wallis H test)
    - Avoid violating strict assumptions to produce reliable results - Normality (Shapiro's Test) and Equality of Variance (Levene's Test)
    - Two approaches - Critical Value and P-Value (Prefered Approach)   
  - To test if there's an statistically significant difference between the mean/median of each group of samples
  - In our use case, it answers the question - Is there a difference in the hourly meso/exp rate between each different map?
  - ```
    # Shapiro-Wilk Test
    tStat, pVal = shapiro(series)

    # Levene's Test
    tStat, pVal = levene(group_one, group_two)

    # Two Sample t-Test (non-equal var) for Meso Group
    tStat_meso, pVal_meso = stats.ttest_ind(a = sf144_meso, b = sf150_meso, equal_var = False, random_state = 5512)

    # Kruskal Wallis Test for Meso Group
    tStat_meso, pVal_meso = stats.kruskal(sf144_meso, sf150_meso,sf158_meso)
    ```

</details>

<details>
  <summary>Posthoc Analysis (scikit-posthoc)</summary>

  - Perform various posthoc analysis (e.g. Dunn's Test, Dunnett's Test, Tukey HSD Test) following a parametric or non-parametric statistical tests
    - Doing pairwise comparison to determine the group of samples with higher mean
  - In our use case, it answers the question - which map has the higher hourly meso/exp rate?
  - ```
    # Dunn's Test for Meso Group
    posthoc_results = sp.posthoc_dunn(a = dftotal_meso, val_col = 'Meso', group_col = 'Map')
    display(posthoc_results)
    ```

</details>


# Analysis and Findings
<details>
  <summary>Assumption of Independence</summary>

  - This assumption should be consider during the design of experiment
  
</details>

<details>
  <summary>Test for Normality</summary>

  - [Shapiro-Wilk Test](https://www.itl.nist.gov/div898/handbook/prc/section2/prc213.htm) - Test for normality (if sample data came from a normally distributed population)
    - Null Hypothesis (H<sub>0</sub>) - your data is from a normally distributed population
    - Alternate Hypothesis (H<sub>A</sub>) - your data is ***not*** from a normally distributed population
  - P-Value Approach
    - if the P-Value is lesser than alpha (0.05), H<sub>0</sub> ***is rejected*** and conclude that the data is ***not normally distributed***
    - else (P-Value >= alpha), H<sub>0</sub> ***is not rejected*** and conclude that the data is ***normally distributed*** 
  
</details>

<details>
  <summary>Test for Equal Variance</summary>

  - [Levene's Test](https://www.itl.nist.gov/div898/handbook/eda/section3/eda35a.htm) - Test for Equal Variance
    - Null Hypothesis (H<sub>0</sub>) - ***Equal*** Variance
    - Alternate Hypothesis (H<sub>A</sub>) - At least one group has ***non-equal*** variance
  - P-Value Approach
    - if the P-Value is lesser than alpha (0.05), H<sub>0</sub> ***is rejected*** and conclude at least one group has ***non-equal variance***
    - else (P-Value >= alpha), H<sub>0</sub> ***is not rejected*** and conclude that each group has ***equal variance*** 

</details>

<details>
  <summary>Choice of Test</summary>

  - If both assumptions are met, we use ***parametric*** test (e.g.t-Test, ANOVA)
  - else, we use ***non-parametric*** test (e.g. X<super>2</super> test, Kruskal-Wallis Test, welch t-test)

</details>

<details>
  <summary>Testing of Two Groups</summary>

  - Used to compare the means between ***TWO*** groups
  - Two Sample Independent t-Test (parametric) & welch t-Test (Non-parametric)
  - SF144 vs. SF150 - Passed the check for Normality
    - Meso Group - Non-Equal Variance (use Welch t-Test)
      - Conclusion: Equal mean between groups (average meso/hr ***don't differ***)   
    - Exp Group - Equal Variance (use Two sample independent t-Test)
      - Conclusion: Non-Equal mean between groups (average xp/hr ***differs***) 
  - SF150 vs. SF158 - Passed Normality 
    - Meso Group - Equal Variance (use Two sample independent t-Test)
      - Conclusion: Non-Equal mean between groups (average meso/hr ***differs***) 
    - Exp Group -  Non-Equal Variance (use Welch t-Test)
      - Conclusion: Non-Equal mean between groups (average xp/hr ***differs***) 
  
  💡There is no need to compare SF144 vs. SF158 as it follows that SF158 is superior to SF150, which is superior to SF144
</details>


<details>
  <summary>Testing of Three Groups</summary>

  - Used to compare the means between ***Three or more*** groups
  - ANOVA (parametric) & Kruskal Wallis Test (Non-parametric)
  - SF144 vs. SF150 vs. SF158 - Passed Normality assumption
    - Meso Group - Non-Equal Variance (use Kruskal Wallis H Test)
      - Conclusion: Non-Equal mean between groups (average meso/hr ***differs***) 
    - Exp Group - Non-Equal Variance (use Kruskal Wallis H Test)
      - Conclusion: Non-Equal mean between groups (average xp/hr ***differs***)
     
  - Following Kruskal Wallis Test, dunn's test is conducted

    - <img src="https://github.com/user-attachments/assets/b1b651df-db7a-4515-9867-8043bf4f1de3" width="350" height="200"/>

    - <img src="https://github.com/user-attachments/assets/ba0e8503-500c-4896-bda2-aa5ebda238a3" width="350" height="200"/>

</details>

<details>
  <summary>Findings and Conclusion</summary>

  <details>
    <summary>SF144 vs. SF150</summary>
    
TL;DR - Pick <ins>***SF144 for meso only***</ins>; else SF150

  A two-sample t-test was conducted to compare the hourly average meso and experience rates between SF144 and SF150. The results indicate that there is no statistically significant difference in the hourly average meso rate between the two maps. However, a statistically significant difference was found in the hourly average experience rate.

Preliminary analysis revealed a slight decrease in the hourly average meso rate (-2.54%) on SF150 compared to SF144. Conversely, a notable increase in the hourly average experience rate (+9.81%) was observed on SF150.

Given the objective of maximizing overall resource gain, <ins>SF150 is preferred</ins>. The relatively small, statistically insignificant decrease in meso rate is outweighed by the significant increase in experience rate. This conclusion is further supported by the results of the t-test, which confirm that the difference in meso rates is likely due to chance, while the difference in experience rates is statistically significant.
  
  </details>

  <details>
    <summary>SF150 vs. SF158</summary>
  
TL;DR - Pick <ins>***SF158***</ins> over SF150  
  
  A two-sample t-test was conducted to compare the hourly average meso and experience rates between SF150 and SF158. The results indicate that there is a statistically significant difference in both the hourly average meso rate and the hourly average experience rate between both maps.


Preliminary analysis revealed a substantial increase in both the hourly average meso rate (31.74%) and the hourly average experience rate (24.32%) on SF158 compared to SF150.

Given the objective of maximizing overall resource gain, <ins>SF158 is preferred</ins>. The statistically significant increases in both meso and experience rates on SF158 make it a superior choice. This conclusion is further supported by the results of the t-test, which confirm that the differences in both meso and experience rates are not due to chance.
  
  </details>

  <details>
    <summary>SF144 vs. SF150 vs. SF158</summary>

TL;DR - Pick <ins>***SF158***</ins> over all the other options  
    
  A one-way Kruskal-Wallis test was conducted to compare the hourly average meso and experience rates among SF144, SF150, and SF158. The results indicate that at least one group has a significantly different mean compared to the others.

Preliminary analysis revealed that SF158 has higher hourly average meso and experience rates compared to both SF144 and SF150.

To determine specific pairwise differences, a post-hoc Dunn's test was conducted. For the meso rate, significant differences were found between SF144 and SF158, and between SF150 and SF158. For the experience rate, significant differences were found between all pairwise comparisons.

Based on these findings, <ins>SF158 is the superior map</ins> in terms of both meso and experience gain. This conclusion is further supported by the previous t-test results.
  
  </details>
  
  <details>
    <summary>Future Works & Limitations</summary>

One area worth exploring is to extend the boundry of analysis to higher starforce maps (e.g. SF168, SF180,SF184) or Arcane Force Maps (AF60, AF90) to make meaningful comparisons.

  
  </details>
  
</details>
