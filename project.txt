from PIL import Image, ImageTk
from skimage.color import rgb2gray
from skimage import img_as_ubyte
import numpy as np
from scipy import signal
from skimage.morphology import binary_closing, binary_opening, disk, convex_hull_image
import matplotlib.pylab as pylab
import tkinter as tk

# 이미지 가져오기 
im1_1 = Image.open("./apple1.jpg")
im1_2 = im1_1.copy()


# Prewitt Operator
imGray = rgb2gray(np.array(im1_2))
kernelX = np.array([ [1, 0, -1], [1, 0, -1], [1, 0, -1] ], dtype=np.float64)*(1.0/6.0)
kernelY = np.array([ [1, 1, 1], [0, 0, 0], [-1, -1, -1] ], dtype=np.float64)*(1.0/6.0)
imFilterX = signal.convolve2d(imGray, kernelX, mode='same')
imFilterY = signal.convolve2d(imGray, kernelY, mode='same')
imMagnitude_prewitt = np.sqrt(imFilterX**2 + imFilterY**2)

# Prewitt Threshold
threshold_prewitt = 0.10
imMagnitude_prewitt1 = imMagnitude_prewitt.copy()
imMagnitude_prewitt1[imMagnitude_prewitt1 < threshold_prewitt], imMagnitude_prewitt1[imMagnitude_prewitt1 >= threshold_prewitt] = 0 , 1


# closing  
im1_closing = binary_closing(imMagnitude_prewitt1, disk(10))

# convex hole 
#im1_convex = im1_closing.copy()
#im1_convex = 1 - im1_convex 
#im1_convex = convex_hull_image(im1_convex)

# reverse_closing
#im1_rev = im1_closing.copy()
#im1_rev = 1 - im1_rev
#im1_rev = binary_closing(im1_rev, disk(15))

# opening 
im1_opening = im1_closing.copy()
im1_opening = binary_opening(im1_opening, disk(15))
im1_opening = 1 - im1_opening


# mask 
foremask1 = np.array(im1_1.copy())
mask1 = im1_opening.copy().astype(int)
im1_find = np.where(mask1 == 0)
#im1_set = list(zip(im1_find[0], im1_find[1]))
foremask1[im1_find] = [0,0,0]

im1_mask1 = Image.fromarray(mask1)

area = 0
for (x, y) in [ (i,j) for i in range(300) for j in range(300)]:
    area += im1_mask1.getpixel((x,y))
print(area)
# color
im1_color = Image.fromarray(foremask1)
im1_r, im1_g, im1_b = im1_color.split()

# subplot 
pylab.gray()
pylab.subplot(251), pylab.imshow(im1_1), pylab.title('input', size=10) , pylab.axis('off')
pylab.subplot(252), pylab.imshow(imGray), pylab.title('imGray', size=10), pylab.axis('off')
pylab.subplot(253), pylab.imshow(imMagnitude_prewitt), pylab.title('prewitt', size=10) , pylab.axis('off')
pylab.subplot(254), pylab.imshow(imMagnitude_prewitt1), pylab.title('threshold', size=10) , pylab.axis('off')
pylab.subplot(255), pylab.imshow(im1_closing), pylab.title('closing', size=10) , pylab.axis('off')
pylab.subplot(256), pylab.imshow(im1_opening), pylab.title('opening', size=10) , pylab.axis('off')
pylab.subplot(257), pylab.imshow(foremask1), pylab.title('mask', size=10) , pylab.axis('off')
pylab.subplot(258), pylab.imshow(im1_r), pylab.title('red', size=10) , pylab.axis('off')
pylab.subplot(259), pylab.imshow(im1_g), pylab.title('green', size=10) , pylab.axis('off')
pylab.subplot(2,5,10), pylab.imshow(im1_b), pylab.title('blue', size=10) , pylab.axis('off')


# TK UI 설정 
root = tk.Tk()   
root.title("road edge detection")


# 가로 세로 정의 
w, h = 300, 300 


# 프레임 정의 
frame0 = tk.Frame(root, width=w, height=h) 
frame0.grid(row=0, column=0, padx=5, pady=5)

frame1 = tk.Frame(root, width=w, height=h) 
frame1.grid(row=0, column=1, padx=5, pady=5)

frame2 = tk.Frame(root, width=w, height=h) 
frame2.grid(row=0, column=2, padx=5, pady=5)


# TK 이미지 정의 
img_color = ImageTk.PhotoImage(image = Image.fromarray(np.uint8(im1_2)))
img_gray = ImageTk.PhotoImage(image = Image.fromarray(np.uint8(im1_color.convert('L')))) 
img_red = ImageTk.PhotoImage(image = Image.fromarray(np.uint8(im1_r)))
img_green = ImageTk.PhotoImage(image = Image.fromarray(np.uint8(im1_g)))
img_blue = ImageTk.PhotoImage(image = Image.fromarray(np.uint8(im1_b))) 
img_mask = ImageTk.PhotoImage(image = Image.fromarray(mask1.astype(bool))) 
img_foremask = ImageTk.PhotoImage(image = im1_color)

# 라벨 정의 
label0 = tk.Label(frame0, text="Input Image")
label0.pack() 

label1 = tk.Label(frame1, text="Foreground")
label1.pack() 

label2 = tk.Label(frame2, text="color analysis")
label2.pack() 

label3 = tk.Label(frame0, text="넓이 = " + str(area)+ " pixel")
label3.pack(side='bottom') 

# 켄버스 정의 
canvas0 = tk.Canvas(frame0, width=w, height=h)
canvas0view = canvas0.create_image(0,0,anchor="nw", image=img_color)
canvas0.pack()  

canvas1 = tk.Canvas(frame1, width=w, height=h)
canvas1view = canvas1.create_image(0,0,anchor="nw", image=img_mask)
canvas1.pack()  

canvas2 = tk.Canvas(frame2, width=w, height=h)
canvas2view = canvas2.create_image(0,0,anchor="nw", image=img_gray)
canvas2.pack()  

# 이벤트 함수 정의 
def fore_mask():
    canvas1.itemconfig(canvas1view, image=img_foremask)

def mask_image():
    canvas1.itemconfig(canvas1view, image=img_mask)

def gray():
    canvas2.itemconfig(canvas2view, image=img_gray)

def red():
    canvas2.itemconfig(canvas2view, image=img_red)
    
def green():
    canvas2.itemconfig(canvas2view, image=img_green)

def blue():
    canvas2.itemconfig(canvas2view, image=img_blue)


# 버튼 설정 
button0_1 = tk.Button(frame1, text="foremask", command=fore_mask)
button0_1.pack(side='right')
button0_2 = tk.Button(frame1, text="mask", command=mask_image)
button0_2.pack(side='right')
button1_4 = tk.Button(frame2, text="blue", command=blue)
button1_4.pack(side='right')
button1_3 = tk.Button(frame2, text="green", command=green)
button1_3.pack(side='right')
button1_2 = tk.Button(frame2, text="red", command=red)
button1_2.pack(side='right')
button1_1 = tk.Button(frame2, text="gray", command=gray)
button1_1.pack(side='right')


# 반복 
root.mainloop()




