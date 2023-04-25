# HandTrackingCalculator
ハンドトラッキングを使って簡単な計算ができるプログラム
from ast import Num
import numbers
from tkinter import N
import cv2
import mediapipe as mp

mp_drawing = mp.solutions.drawing_utils
mp_hands = mp.solutions.hands

video = cv2.VideoCapture(0)

#入力した数字
num = 0

#以下の"o数字"は読み込んだ直近9回の数字を記録する変数。初期状態は指で入力できない0.1とした
o1 = 0.1
o2 = 0.1
o3 = 0.1
o4 = 0.1
o5 = 0.1
o6 = 0.1
o7 = 0.1
o8 = 0.1
o9 = 0.1

#各ステップを検知するフラグ
flag0 = 0 #リセットコマンドが使用されたら１になるフラグ
flag1 = 0 #一つ目の数字が入力されたら１に、入力内容を表示したら２になるフラグ
flag2 = 0 #一つ目の数字が確定されたら１に、計算記号を決定したら２になるフラグ
flag3 = 0 #計算記号が決定されると１に、二つ目の数字が入力されたら２に、入力内容を表示したら３になるフラグ
flag4 = 0 #二つ目の数字が確定されたら１に、計算を実行したら２になるフラグ
flag5 = 0 #計算を実行する時に１になるフラグ
flag6 = 0 #計算結果を表示したら１になり、以降リセットコマンドを使用するまで操作を受け付けなくするフラグ

#計算記号を記録する変数。初期状態であるか否かで分岐したいため初期状態を定義する
box2 = 0

continue_message = "\n(please LEFT FIST for CONTINUE!!!)"

with mp_hands.Hands(
    model_complexity=0,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5) as hands:
    while True:
        ret,image=video.read()
        image=cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        image.flags.writeable=False
        results=hands.process(image)
        image.flags.writeable=True
        image=cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
        lmList=[]
        if results.multi_hand_landmarks:
            for hand_landmark in results.multi_hand_landmarks:
                myHands=results.multi_hand_landmarks[0]
                for id, lm in enumerate(myHands.landmark):
                    h,w,c=image.shape
                    cx,cy = int(lm.x*w), int(lm.y*h)
                    lmList.append([id,cx,cy])
                mp_drawing.draw_landmarks(image, hand_landmark, mp_hands.HAND_CONNECTIONS)
            if len(lmList)!=0:

                #左右を親指の位置で判別する
                
                if lmList[5][1] > lmList[13][1]:
                    num0 = 1
                elif lmList[5][1] < lmList[13][1]:
                    num0 = -1

                #数字（指を用いた二進法）
        
                if (lmList[4][1])*num0 > (lmList[2][1])*num0:
                    num1 = 1
                else:
                    num1 = 0

                if lmList[8][2] < lmList[6][2]:
                    num2 = 1
                else:
                    num2 = 0

                if lmList[12][2] < lmList[10][2]:
                    num3 = 1
                else:
                    num3 = 0
                
                if lmList[16][2] < lmList[14][2]:
                    num4 = 1
                else:
                    num4 = 0

                if lmList[20][2] < lmList[18][2]:
                    num5 = 1
                else:
                    num5 = 0

                num = ((num1*1) + (num2*2) + (num3*4) + (num4*8) + (num5*16)) * num0

                if flag5 == 1:
                    if box2 == 1:
                        result = box1 + box3
                        symbol = "+"
                    elif box2 == 2:
                        result = box1 - box3
                        symbol = "-"
                    elif box2 == 3:
                        symbol = "*"
                        result = box1 * box3
                    elif box2 == 4:
                        symbol = "/"
                        result = box1 / box3
                    elif box2 == 5:
                        symbol = "**"
                        result = box1 ** box3

                    if flag6 == 0:
                        print(str(box1) + str(symbol) + str(box3) + "=" + str(result) + str(continue_message))
                        flag6 = 1
                        flag5 = 0

                    

                elif flag5 == 0:

                    if num0 == -1:
                        minum = num*-1
                        if flag2 == 1:
                            if minum == 2:
                                box2 = 1
                                print("add +")
                            elif minum == 6:
                                box2 = 2
                                print("subtraction -")
                            elif minum == 14:
                                box2 = 3
                                print("multiplication *")
                            elif minum == 30:
                                box2 = 4
                                print("division /")
                            elif minum == 31:
                                box2 = 5
                                print("power **")

                            if box2 != 0:
                                flag2 = 2
                                flag3 = 1

                        elif flag4 == 1:
                            if minum == 31:
                                print("OK!")
                                flag4 = 2
                                flag5 = 1

                        
                        if (minum == 0) and (flag0 == 0):
                            flag0 = 1
                            flag1 = 0
                            flag2 = 0
                            flag3 = 0
                            flag4 = 0
                            flag5 = 0
                            flag6 = 0
                            box1 = 0
                            box2 = 0
                            box3 = 0
                            if flag0 == 1:
                                print("reset")
                        
                    

                
                    else: #numが正のとき（右手のとき）
                        if (o1 == num) and (o2 == num) and (o3 == num) and (o4 == num) and (o5 == num) and (o6 == num) and (o7 == num) and (o8 == num) and (o9 == num) and flag1 == 0:
                            w1 = "first number is "
                            w2 =  "."
                            flag1 = 1
                            box1 = num

                        elif (o1 == num) and (o2 == num) and (o3 == num) and (o4 == num) and (o5 == num) and (o6 == num) and (o7 == num) and (o8 == num) and (o9 == num) and flag3 == 1:
                            w1 = "second number is "
                            w2 = "."
                            flag3 = 2
                            box3 = num


                        if o9 == 0.1:
                            print("Loading...")
                
                
                        elif (o1 != num) and (o2 != num) and (o3 != num) and (o4 != num) and (o5 != num) and (o6 != num) and (o7 != num) and (o8 != num) and (o9 != num) and flag6 == 0:
                            print(num)
                            flag0 = 0
                            
                        elif (flag1 == 1) or (flag3 == 2):
                            print(str(w1)+str(num)+str(w2))
                            if flag3 == 2:
                                flag3 = 3
                                flag4 = 1
                            elif flag1 == 1:
                                flag1 = 2
                                flag2 = 1
                                
                    o9 = o8
                    o8 = o7
                    o7 = o6
                    o6 = o5
                    o5 = o4
                    o4 = o3
                    o3 = o2
                    o2 = o1
                    o1 = num
                    
            cv2.imshow("Frame",image)
            k=cv2.waitKey(1)
            if k==ord('q'):
                break
    video.release()
    cv2.destroyAllWindows()
