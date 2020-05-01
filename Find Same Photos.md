# 중복 사진정리

1년 넘게 미룬 사진정리를 드디어 해야겠다고 마음먹었다. 내 사진첩에는 중복된 사진이 너무나도 많았는데 하나하나 손으로 할 수 없다고 생각했다. 그래서 python에 있는 이미지 인식을 이용하였다.

수천개의 사진의 이미지를 전부 대조할 수 없으니 우선적으로 처리할 수 있는 방안을 생각해보았다.  
&nbsp;   
    
    💡생각 1 : 저장명이 겹치면 같은 파일이다. 

    ex) 
      6E5DD712-4FDF-4677-AF33-63E29F975035.jpeg
      6E5DD712-4FDF-4677-AF33-63E29F975035 2.jpeg
 
    💡생각 2 : 용량이 같으면 같은 파일이다. 
    
&nbsp;    
이 두가지 생각을 토대로 중복사진 정리를 해보았다.



```python
import os   # 운영체제
import pandas as pd   # 판다스
```

용량이 같고 이름이 겹치면 같은 파일이다.  
ex)  
&nbsp; 6E5DD712-4FDF-4677-AF33-63E29F975035.jpeg  
&nbsp; 6E5DD712-4FDF-4677-AF33-63E29F975035 2.jpeg  
  
1. 용량이 같은 것끼리 분류
2. 같은 용량끼리 이미지 비교
---  

### Photo File

1. 사진 목록 불러오기  
```os.listdir('path')``` : 경로에 있는 파일 목록 불러오기


```python
photo_list = []
for f in os.listdir('/Users/mizy/Desktop/mizy/Image/Photo'):
    if 'jpeg' in f:
        photo_list.append(f)
```

2. 사진의 크기 구하기  
```os.path.getsize('path/filename')``` : 파일 크기 구하기


```python
photo_size = list(map(lambda x: os.path.getsize('/Users/mizy/Desktop/mizy/Image/Photo' + '/' + x), photo_list))
```

3. 데이터프레임으로 만들기


```python
# Find Same Photos
fsp = pd.DataFrame({'filename_raw':photo_list, 'size':photo_size})
```


```python
print('사진의 갯수 :', len(fsp))
```

    사진의 갯수 : 5081



```python
fsp.sample(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filename_raw</th>
      <th>size</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>4788</td>
      <td>1DCBB19B-D671-4293-AE4C-44E0BF3EACF1 2.jpeg</td>
      <td>451001</td>
    </tr>
    <tr>
      <td>3845</td>
      <td>05FDA157-8781-4C5E-9828-54E3DBF3388E.jpeg</td>
      <td>471993</td>
    </tr>
  </tbody>
</table>
</div>



'filename 숫자.jpeg' 형식으로 이루어진 사진 이름이 있는데 < 💡생각1 >에서 말한대로 저장명이 겹치고 맨 뒤에 ' 숫자'가 오면 같은 파일이라고 볼 수 있다. 따라서 ' 숫자'를 제거하여 파일명이 중복인 사진을 구분할 것이다. 

4. &nbsp;    
&nbsp; ' 숫자' 제거하기  


```python
import re   # 정규표현식

com = re.compile(' \d')
fsp['filename'] = list(map(lambda x: com.sub('', x), photo_list))
```

정규표현식을 이용하여 ' 숫자'에 해당하는 값을 지워주었다. 

### 겹치는 파일명 / 파일크기 확인


```python
# Photo Value Counts
pvc = pd.DataFrame({'filename':fsp['filename'].value_counts().index, 'fn_counts':fsp['filename'].value_counts().values})   
psvc = pd.DataFrame({'size':fsp['size'].value_counts().index, 'size_counts':fsp['size'].value_counts().values})   

fsp = pd.merge(fsp, pvc, how = 'left', on = 'filename')
fsp = pd.merge(fsp, psvc, how = 'left', on = 'size')
```

각 파일별로 겹치는 이름과 크기가 몇 개씩인지 확인해보았다.


```python
fsp.sample(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filename_raw</th>
      <th>size</th>
      <th>filename</th>
      <th>fn_counts</th>
      <th>size_counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1828</td>
      <td>FAEECB8C-184D-40E7-AF54-5E802724D338.jpeg</td>
      <td>143629</td>
      <td>FAEECB8C-184D-40E7-AF54-5E802724D338.jpeg</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <td>3015</td>
      <td>323323A4-2BF2-4F7F-92FC-65EE52D75E69.jpeg</td>
      <td>703598</td>
      <td>323323A4-2BF2-4F7F-92FC-65EE52D75E69.jpeg</td>
      <td>2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



🙋🏻‍♀️Q. 이름은 같은데 크기가 다른게 있나?


```python
for i in range(len(fsp)):
    if (fsp['fn_counts'][i] > 1) & (fsp['size_counts'][i] == 1):
        print(i)
```

→ 없다. 이름이 같으면 크기도 같음  
= 이름이 같은건 하나만 남겨도 됨


```python
# Find Same Phto_Not Same Name
fsp_nsn = fsp.sort_values(['filename_raw'], ascending = False).drop_duplicates(['filename'], keep = 'first')
```

```df.sort_values(['기준열'])``` : 정렬  
```df.drop_duplicates(['기준열'])``` : 중복제거

' 숫자'인 파일을 제거하고 싶어서 내림차순으로 정렬해주었다. 이후 숫자가 제거된 파일명을 기준으로 중복제거하였다. 


```python
print('남은 사진의 갯수 : {}\n지워진 사진의 갯수 : {}'.format(len(fsp_nsn), len(fsp)-len(fsp_nsn)))
```

    남은 사진의 갯수 : 3877
    지워진 사진의 갯수 : 1204


---  


```python
pvc_nsn = pd.DataFrame({'filename':fsp_nsn['filename'].value_counts().index, 'fn_counts_nsn':fsp_nsn['filename'].value_counts().values})   
psvc_nsn = pd.DataFrame({'size':fsp_nsn['size'].value_counts().index, 'size_counts_nsn':fsp_nsn['size'].value_counts().values})   

fsp_nsn = pd.merge(fsp_nsn, pvc_nsn, how = 'left', on = 'filename')
fsp_nsn = pd.merge(fsp_nsn, psvc_nsn, how = 'left', on = 'size')
```

🙋🏻‍♀️ Q. 이름 겹치는게 남아있나?


```python
fsp_nsn[fsp_nsn['fn_counts_nsn']!=1]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>filename_raw</th>
      <th>size</th>
      <th>filename</th>
      <th>fn_counts</th>
      <th>size_counts</th>
      <th>fn_counts_nsn</th>
      <th>size_counts_nsn</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



→ 없다.  

🙋🏻‍♀️ Q. 남아있는 사이즈 겹치는 것들의 갯수는?


```python
print('사이즈 겹치는 사진의 갯수 :', len(fsp_nsn[fsp_nsn['size_counts_nsn']!=1]))
print('중복 사이즈의 갯수 :', len(psvc_nsn[psvc_nsn['size_counts_nsn']>1]))
```

    사이즈 겹치는 사진의 갯수 : 60
    중복 사이즈의 갯수 : 30


### 이미지 비교

남은 것은 사이즈는 같지만 저장명은 완전히 다른 것들이다. 이미지 비교가 필요한 사진들이라 OpenCV와 skimage를 이용하였다.

&nbsp; 1) 사이즈가 같은 두 이미지를 불러온다.  
&nbsp; 2) 이미지를 array로 변환했을 때 구조가 같다면 두 이미지에 차이가 있는지 확인한다.   
&nbsp; 3) 만약 두개의 이미지 구조가 같은데 차이가 존재한다면 그 사진은 직접 확인하기로 한다.  

위 순서대로 아래 코드를 작성해보았다.  

```cv2.imread('path')``` : 이미지 읽기


```python
import cv2   # OpenCV
from skimage.measure import compare_ssim

# 삭제될 사진의 리스트
delete = []


for i in range(len(psvc_nsn)):
    
    # 중복된 크기(size)가 있는 경우
    if psvc_nsn['size_counts_nsn'][i] == 2:
        
        # 그 크기에 해당하는 사진을 불러온다. 
        temp = fsp_nsn[fsp_nsn['size']==psvc_nsn['size'][i]].reset_index(drop = True).sort_values(['filename'])
        
        # 사진 읽기
        imageA = cv2.imread('Photo/'+temp['filename_raw'][0])
        imageB = cv2.imread('Photo/'+temp['filename_raw'][1])
        
        # 이미지를 grayscale로 변환
        grayA = cv2.cvtColor(imageA, cv2.COLOR_BGR2GRAY)
        grayB = cv2.cvtColor(imageB, cv2.COLOR_BGR2GRAY)
        
        # 이미지의 구조가 같다면 이미지 비교
        if len(grayA)==len(grayB):
            (score, diff) = compare_ssim(grayA, grayB, full=True)
            
            # 차이가 없다면 하나는 delete에 넣어주기
            if score == 1:
                delete.append(temp['filename_raw'][1])
            
            # 구조가 같지만 차이가 존재한다면 직접 확인하기     
            else:
                print('확인해보시오! : ', temp['filename_raw'][0], '/', temp['filename_raw'][1], f'(score : {score})')
```

    확인해보시오! :  CB8D02A7-A4CD-4F8C-9459-DDB7D57BB5B2.jpeg / B72DC723-5AC4-42FE-8A46-CB68A16123D3.jpeg (score : 0.33035451451666076)
    확인해보시오! :  A1D66410-97E2-4E50-A605-47D589270D07.jpeg / 2A2FDD14-AE0B-4C93-89CA-F440646BBF8B.jpeg (score : 0.9999543491150039)


### Result

1. Delete  
앞전에 사진명 중복제거로 제거된 파일들을 delete리스트에 넣어준다.


```python
# 중복제거된 것들은 delete 리스트에 넣어주기
delete = delete + list(fsp[~fsp['filename_raw'].isin(fsp_nsn['filename_raw'])]['filename_raw'])

print('삭제할 목록 :', len(delete))
```

    삭제할 목록 : 1226


2. Result  
전체데이터 - delete = 남길데이터


```python
# result : 처음(fsp)데이터에서 - delete를 제외한 것
result = list(fsp[~fsp['filename_raw'].isin(delete)]['filename_raw'])

print('남길 목록 : ', len(result))
```

    남길 목록 :  3855


3. 사진 옮기기  
현재 정리되지 않은 사진들은 Photo에 있다. Delete와 Result 폴더를 새로 만들어 주어 버릴 사진과 남길 사진을 구분지어 두었다.  
```shutil.move('path/file', '이동할 경로')``` : 파일을 '이동할 경로'로 옮겨준다.  


```python
import shutil

for i in result:
    shutil.move('Photo/'+i, '/Users/mizy/Desktop/mizy/Image/Result')
    
for i in delete:
    shutil.move('Photo/'+i, '/Users/mizy/Desktop/mizy/Image/Delete')    
```

정리 끝!
