MapleStory Mobile (MSM) is a mobile MMORPG inspired by the popular MMORPG, Maplestory, by Nexon. In essence, it's a game about gathering resources to enhance equipment and conquer challenging endgame content. To maximize resource accumulation, players often seek to optimize their gameplay. The introduction of the Auto-Battle (AB) system in MSM has provided a new opportunity for data-driven decision-making.

The EasyOCR library is well-suited for this task due to its simplicity. Our use case involves scanning screenshots of AB sessions to extract relevant statistics for future analysis. Given the consistent nature of the screenshots, we anticipate minimal variation in extraction results.

![ab_08_sf144](https://github.com/user-attachments/assets/b74899d5-71f0-4862-ba1d-935080a2a641)

Figure 1.0 - Example screenshot of a Auto-Battle session

*Note - The important statistics include _duration, kill count, meso gained, red meso gained and total exp gained_.

The EasyOCR library scans through each screenshot to extract the important statistics. Extra  measures are taken to validate and rectify erroneous extraction results. These results are then stored into dataframe for further data processing. 

![sample_image](https://github.com/user-attachments/assets/080ca1ed-b0fa-4924-afe8-595289557182)

Figure 1.1 - Extracted statistic (highlighted in green boxes) from Auto-Battle session

<details>
  <summary>Python libraries are used in our use case  </summary>
    EasyOCR, PLI, Numpy, Pandas, matplotlib, cv2, glob
</details>

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
  - In some cases, we observe that the duration statistic is invalid as the colon has been misread as a period by the EasyOCR library. To ensure the accuracy our result, the above line of code replaces the any punctuation found in the duration statistic with a colon. 

</details>




