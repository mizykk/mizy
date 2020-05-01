# Mizy  

[📸](https://github.com/mizykk/Mizy/blob/master/Find_Same_Photos.ipynb) Find_Same_Photos : 중복되는 사진정리    

---  
### 🐰 python 🐰   
`len()` : 길이     
`append()` : 리스트에 값 추가하기     

  
### 🐼 Pandas 🐼     
`import pandas as pd`   
`pd.DataFrame()` : 데이터프레임 만들기    
`pd.merge(df1, df2, by = , how = )` : 데이터프레임 합치기    
`df.sort_values(['기준열'])` : 정렬  
`df.drop_duplicates(['기준열'])` : 중복제거  
`df.value_counts(['기준열'])` : 열 값의 개수세기  


### 🦊 re 🦊     
`import re`   
`re.compile()`  
`re.sub()` : 문자열 변경하기  


### 🐶 os 🐶    
`import os`  
`os.listdir('path')` : 경로에 있는 파일 목록 불러오기   
`os.path.getsize('path/filename')` : 파일 크기 구하기  


### 🦄 OpenCV 🦄    
`import cv2`  
`cv2.imread('path')` : 이미지 읽기   
`cv2.cvtColor(imageA, cv2.COLOR_BGR2GRAY)` : grayscale로 변환    


### 🐹 And more.. 🐹   
이미지 비교  
`from skimage.measure import compare_ssim`    
`compare_ssim(image1, image2, full=True)`  


파일 옮기기  
`import shutil`  
`shutil.move('path/file', '이동할 경로')` : 파일을 '이동할 경로'로 옮겨준다.  
