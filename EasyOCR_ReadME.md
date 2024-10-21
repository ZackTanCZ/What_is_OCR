MapleStory Mobile (MSM) is a mobile MMORPG inspired by the popular MMORPG, Maplestory, by Nexon. In essence, it's a game about gathering resources to enhance equipment and conquer challenging endgame content. To maximize resource accumulation, players often seek to optimize their gameplay. The introduction of the Auto-Battle (AB) system in MSM has provided a new opportunity for data-driven decision-making.

The EasyOCR library is well-suited for this task due to its simplicity. Our use case involves scanning screenshots of AB sessions to extract relevant statistics for future analysis. Given the consistent nature of the screenshots, we anticipate minimal variation in extraction results.

![ab_08_sf144](https://github.com/user-attachments/assets/b74899d5-71f0-4862-ba1d-935080a2a641)

Figure 1.0 - Example screenshot of a Auto-Battle session

*Note - The important statistics include _duration, kill count, meso gained, red meso gained and total exp gained_.
The EasyOCR library scans through each screenshot to extract the important statistics. Extra  measures are taken to validate and rectify erroneous extraction results. These results are then stored into dataframe for further data processing. 

![sample_image](https://github.com/user-attachments/assets/080ca1ed-b0fa-4924-afe8-595289557182)


The following libraries are used in our use case
  1. EasyOCR
  2. PLI
  3. Numpy
  4. Pandas
  5. matplotlib
  6. cv2
  7. glob





