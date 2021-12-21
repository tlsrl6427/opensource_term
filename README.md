# Minesweeper supporter

<br>

## Description
> 구글 지뢰찾기를 도와주는 프로그램입니다

<br>

## Scenario
1. 구글 지뢰찾기 실행 후 플레이
2. 막히는 곳이 있을때 프로그램 실행
3. 마우스가 위치한 곳이 지뢰위치!

<br>

## Code explannation

* __Library__
    ```python
    import pyautogui as pag
    import keyboard
    import time
    import numpy
    from PIL import Image
    from PIL import ImageGrab
    ```
    > pyautogui: 키보드와 마우스를 제어하는 라이브러리<br>
    > keyboard: 키보드를 제어한느 라이브러리<br>
    > PIL(Python Imaging Library): 이미지를 처리해주는 라이브러리<br>
    > ImageGrab: 화면을 캡쳐하게 해주는 라이브버리
    
   <br>
* __준비물__
  * 좌표찍기
  ```python
  while True:
      if keyboard.is_pressed("F4"): # F4 누른게 감지되면
          t1 = pag.position() # 위치 뽑아서 저장
          print(t1)
          time.sleep(0.5)
          break    

  while True:
      if keyboard.is_pressed("F4"): 
          t2 = pag.position()
          print(t2)
          time.sleep(0.5)
          break  
  ```
  > F4를 누르면 마우스 위치가 좌표로 나타난다

  > 사진의 왼쪽 위와 오른쪽 아래에 마우스를 대고 좌표를 얻는다

  <br>

  * 게임화면 스크린샷 뽑기
  ```python
  region=(t1[0], t1[1], t2[0]-t1[0], t2[1]-t1[1])
  pag.screenshot("C:/Users/tlsrl/opensource_term/window.png", region=region)
  ```
  > 위에서 얻은 좌표를 사진으로 저장한다

  > 저장하는 요소는 게임창과 내부 요소(1,2,3... 등의 숫자들과 flag)
 
 <br>
 
* __진행과정__

  - 게임 창 찾기
  
  ```python
  for i in numpy.arange(0.51,0.99, 0.02):
      p_list = pag.locateAllOnScreen("C:/Users/tlsrl/opensource_term/window.png", confidence=i)
      p_list = list(p_list)
      if len(p_list) == 1:
          print(i)
          print(p_list)
          break
  ```
  > 준비물에 있는 게임 window를 찾는다.
  
  > 신뢰도를 조절해가며 창이 1개만 나올때까지 for문을 돌린다.
  
  <br>
  
  - 게임요소를 행렬화
  
   ```python
  pixel = p_list[0][2]// 18
  col = p_list[0][2] // pixel
  row = (p_list[0][3] - pixel*2) // pixel
  matrix = numpy.zeros((row, col))
  print(matrix)

  ctr_list = []
  for i in range(row):
      ctr_row = []
      for k in range(col):
          ctr_row.append([p_list[0][0] + pixel//2 + k*pixel, p_list[0][1]+ pixel*2 + pixel//2 + i*pixel])
      ctr_list.append(ctr_row)

  ctr_list
   ```
   > matrix는 게임 화면, ctr_list는 각 요소의 중앙 좌표값의 리스트이다
   
   <br>
   
   - 뽑은 사진과 똑같은 부분 찾기
   ```python
    all_pic_list = []

    pic_list_1 = pag.locateAllOnScreen("C:/Users/tlsrl/opensource_term/one.png", confidence=0.70)
    pic_list_1 = list(pic_list_1)
    all_pic_list.append([1, pic_list_1])

    pic_list_2 = pag.locateAllOnScreen("C:/Users/tlsrl/opensource_term/two.png", confidence=0.60)
    pic_list_2 = list(pic_list_2)
    all_pic_list.append([2, pic_list_2])

    pic_list_3 = pag.locateAllOnScreen("C:/Users/tlsrl/opensource_term/three.png", confidence=0.60)
    pic_list_3 = list(pic_list_3)
    all_pic_list.append([3, pic_list_3])

    pic_list_f = pag.locateAllOnScreen("C:/Users/tlsrl/opensource_term/flag.png", confidence=0.7)
    pic_list_f = list(pic_list_f)
    all_pic_list.append([9, pic_list_f])
    ```
    > all_pic_list는 화면의 각 요소의 위치를 담고있는 리스트이다
    
    <br>
    
    - 숫자와 깃발 찾기
    ```python
    for pic_list in all_pic_list:

          ctr = []

          for p in pic_list[1]:
              if not ctr:
                  ctr.append(pag.center(p))
              else:
                  for i in ctr:
                      overlapped = False
                      if abs(i[0]-pag.center(p)[0])<=10 and abs(i[1]-pag.center(p)[1])<=10:
                          overlapped = True
                          break
                  if not overlapped:
                      ctr.append(pag.center(p))
          if pic_list[0] == 9:
              print("flag 의 개수: ", len(ctr))
          else:
              print(pic_list[0], "의 개수: ", len(ctr))

          for i in ctr:
              a = (i[0] - p_list[0][0]) // pixel # col
              b = (i[1] - p_list[0][1] - pixel*2) // pixel # row
              matrix[b][a] = pic_list[0]  
     ```
     > 좌표가 중복되어 나오기 때문에 중복을 지워주고, matrix에 추가한다

     <br>
     
     - 클릭 못하는 곳 지정
      
     ```python
      wall1 = Image.open("C:/Users/tlsrl/opensource_term/wall1.png")
      wall2 = Image.open("C:/Users/tlsrl/opensource_term/wall2.png")
     
      pix1 = numpy.array(wall1)
      pix2 = numpy.array(wall2)
      
      for r in range(row):
          for c in range(col):
              if matrix[r][c]==0:
                  screen = ImageGrab.grab()
                  color_pos = x, y = int(ctr_list[r][c][0]), int(ctr_list[r][c][1])
                  color = screen.getpixel(color_pos)
                  if color[0]==pix1[14][14][0] and color[1]==pix1[14][14][1] and color[2]==pix1[14][14][2]:
                      matrix[r][c] = 99
                  elif color[0]==pix2[14][14][0] and color[1]==pix2[14][14][1] and color[2]==pix2[14][14][2]:
                      matrix[r][c] = 99 
     ```
     > 클릭할 수 없는 숫자 안 공간을 99로 채운다
  - 정답지 만들기
  ```python
  answer_list = numpy.zeros((row, col))
  for r in range(row):
      for c in range(col):
          if matrix[r][c]!= 0 and matrix[r][c]!= 9:
              if 0<=r-1<row and 0<=c-1<col and matrix[r-1][c-1]==0:
                  answer_list[r-1][c-1] +=1
              if 0<=r-1<row and 0<=c<col and matrix[r-1][c]==0:
                  answer_list[r-1][c] +=1
              if 0<=r-1<row and 0<=c+1<col and matrix[r-1][c+1]==0:
                  answer_list[r-1][c+1] +=1
              if 0<=r<row and 0<=c-1<col and matrix[r][c-1]==0:
                  answer_list[r][c-1] +=1
              if 0<=r<row and 0<=c+1<col and matrix[r][c+1]==0:
                  answer_list[r][c+1] +=1
              if 0<=r+1<row and 0<=c-1<col and matrix[r+1][c-1]==0:
                  answer_list[r+1][c-1] +=1
              if 0<=r+1<row and 0<=c<col and matrix[r+1][c]==0:
                  answer_list[r+1][c] +=1
              if 0<=r+1<row and 0<=c+1<col and matrix[r+1][c+1]==0:
                  answer_list[r+1][c+1] +=1
  print("정답")
  print(answer_list)
  ```
  > 0으로 초기화하고 숫자 주위의 빈 곳에 +1을 한다
  
  <br>
  
  - 정답칸에 마우스 이동
  ```python
  answer_r = numpy.argmax(answer_list) // col
  answer_c = numpy.argmax(answer_list) % col
  pag.moveTo(ctr_list[answer_r][answer_c][0], ctr_list[answer_r][answer_c][1], 0.8)
  ```
  > 0.8초의 속도로 정답지 중 가장 큰 곳의 좌표로 마우스를 움직인다
  
<br>

* __결과예시__

    - 게임화면 예시<br>

    ![지뢰찾기 문제](/mine_problem.JPG)

    <br>

    - 행렬화면 예시<br>
    ![지뢰찾기 정답](/mine_answer.JPG)
