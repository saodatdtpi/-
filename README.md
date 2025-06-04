# -
ПРИМЕНЕНИЕ ТЕХНОЛОГИЙ КОМПЬЮТЕРНОГО ЗРЕНИЯ ДЛЯ АНАЛИЗА МЕДИЦИНСКИХ ИЗОБРАЖЕНИЙ
import sys
import cv2
import numpy as np
import tkinter as tk
import tkinter.ttk as ttk
import math
from PIL import Image,ImageTk
from tkinter import filedialog

try:
    import tkinter as tk
except ImportError:
    import tkinter as tk

try:
    import tkinter.ttk
    py3 = False
except ImportError:
    import tkinter.ttk as ttk
    py3 = True

import mam_support

def vp_start_gui():
    '''Starting point when module is the main routine.'''
    global val, w, root
    root = tk.Tk()
    top = Toplevel1 (root)
    mam_support.init(root, top)
    root.mainloop()

w = None
def create_Toplevel1(rt, *args, **kwargs):
    '''Starting point when module is imported by another module.
       Correct form of call: 'create_Toplevel1(root, *args, **kwargs)' .'''
    global w, w_win, root
    #rt = root
    root = rt
    w = tk.Toplevel (root)
    top = Toplevel1 (w)
    mam_support.init(w, top, *args, **kwargs)
    return (w, top)

def destroy_Toplevel1():
    global w
    w.destroy()
    w = None

class Toplevel1:
    def __init__(self, top=None):
        '''This class configures and populates the toplevel window.
           top is the toplevel containing window.'''
        _bgcolor = '#d9d9d9'  # X11 color: 'gray85'
        _fgcolor = '#000000'  # X11 color: 'black'
        _compcolor = '#d9d9d9' # X11 color: 'gray85'
        _ana1color = '#d9d9d9' # X11 color: 'gray85'
        _ana2color = '#ececec' # Closest X11 color: 'gray92'

        top.geometry("1530x790+0+0")
        top.minsize(148, 1)
        #top.maxsize(4104, 1055)
        top.resizable(1,  1)
        top.title("Oбнаружение заболеваний с помощью медицинских изображений")
        top.configure(background="#d9d9d9")

        self.menubar = tk.Menu(top,font="TkMenuFont",bg=_bgcolor,fg=_fgcolor)
        top.configure(menu = self.menubar)

        self.Frame1 = tk.Frame(top)
        self.Frame1.place(relx=0.006, rely=0.011, relheight=0.969
                , relwidth=0.983)
        self.Frame1.configure(relief='groove')
        self.Frame1.configure(borderwidth="2")
        self.Frame1.configure(relief="groove")
        self.Frame1.configure(background="#d9d9d9")
        self.Frame1.configure(cursor="fleur")

        self.Label1 = tk.Label(self.Frame1)
        self.Label1.place(relx=0.014, rely=0.011, height=56, width=1559)
        self.Label1.configure(background="#00cec8")
        self.Label1.configure(disabledforeground="#a3a3a3")
        self.Label1.configure(font="-family {Segoe UI} -size 16 -weight bold -slant italic")
        self.Label1.configure(foreground="#c11c84")
        self.Label1.configure(text='''Улучшение изображения маммограммы, расчет диаметра/площади и оценка стадии опухоли''')

        self.browselabel = tk.Label(self.Frame1)
        self.browselabel.place(relx=0.019, rely=0.114, height=404, width=477)
        self.browselabel.configure(background="#ffffff")
        self.browselabel.configure(disabledforeground="#a3a3a3")
        self.browselabel.configure(foreground="#000000")

        self.enhanceimage = tk.Label(self.Frame1)
        self.enhanceimage.place(relx=0.347, rely=0.114, height=404, width=475)
        self.enhanceimage.configure(activebackground="#f9f9f9")
        self.enhanceimage.configure(activeforeground="black")
        self.enhanceimage.configure(background="#ffffff")
        self.enhanceimage.configure(disabledforeground="#a3a3a3")
        self.enhanceimage.configure(foreground="#000000")
        self.enhanceimage.configure(highlightbackground="#d9d9d9")
        self.enhanceimage.configure(highlightcolor="black")

        self.segImg = tk.Label(self.Frame1)
        self.segImg.place(relx=0.682, rely=0.114, height=404, width=475)
        self.segImg.configure(activebackground="#f9f9f9")
        self.segImg.configure(activeforeground="black")
        self.segImg.configure(background="#ffffff")
        self.segImg.configure(disabledforeground="#a3a3a3")
        self.segImg.configure(foreground="#000000")
        self.segImg.configure(highlightbackground="#d9d9d9")
        self.segImg.configure(highlightcolor="black")

        self.browseimg = tk.Button(self.Frame1)
        self.browseimg.place(relx=0.039, rely=0.604, height=33, width=366)
        self.browseimg.configure(activebackground="#ececec")
        self.browseimg.configure(activeforeground="#000000")
        self.browseimg.configure(background="#d9d9d9")
        self.browseimg.configure(command=self.browse)
        self.browseimg.configure(disabledforeground="#a3a3a3")
        self.browseimg.configure(foreground="#000000")
        self.browseimg.configure(highlightbackground="#d9d9d9")
        self.browseimg.configure(highlightcolor="black")
        self.browseimg.configure(pady="0")
        self.browseimg.configure(text='''Выберите картинку''')

        self.enhance = tk.Button(self.Frame1)
        self.enhance.place(relx=0.373, rely=0.604, height=33, width=366)
        self.enhance.configure(activebackground="#ececec")
        self.enhance.configure(activeforeground="#000000")
        self.enhance.configure(background="#d9d9d9")
        self.enhance.configure(cursor="fleur")
        self.enhance.configure(disabledforeground="#a3a3a3")
        self.enhance.configure(foreground="#000000")
        self.enhance.configure(highlightbackground="#d9d9d9")
        self.enhance.configure(highlightcolor="black")
        self.enhance.configure(pady="0")
        self.enhance.configure(text='''Улучшение изображения''')

        self.segment = tk.Button(self.Frame1)
        self.segment.place(relx=0.723, rely=0.595, height=33, width=366)
        self.segment.configure(activebackground="#ececec")
        self.segment.configure(activeforeground="#000000")
        self.segment.configure(command=self.segment)
        self.segment.configure(background="#d9d9d9")
        self.segment.configure(disabledforeground="#a3a3a3")
        self.segment.configure(foreground="#000000")
        self.segment.configure(highlightbackground="#d9d9d9")
        self.segment.configure(highlightcolor="black")
        self.segment.configure(pady="0")
        self.segment.configure(text='''Сегментированное изображение''')

        self.opening = tk.Label(self.Frame1)
        self.opening.place(relx=0.455, rely=0.729, height=28, width=197)
        self.opening.configure(background="#d9d9d9")
        self.opening.configure(disabledforeground="#a3a3a3")
        self.opening.configure(foreground="#000000")
        self.opening.configure(text='''Открытие:''')

        self.Scale1 = tk.Scale(self.Frame1, from_=0.0, to=50.0)
        self.Scale1.place(relx=0.58, rely=0.707, relwidth=0.145, relheight=0.0
                , height=48, bordermode='ignore')
        self.Scale1.configure(activebackground="#ececec")
        self.Scale1.configure(background="#d9d9d9")
        self.Scale1.configure(foreground="#000000")
        self.Scale1.configure(highlightbackground="#d9d9d9")
        self.Scale1.configure(highlightcolor="black")
        self.Scale1.configure(orient="horizontal")
        self.Scale1.configure(troughcolor="#d9d9d9")

        self.Label2 = tk.Label(self.Frame1)
        self.Label2.place(relx=0.487, rely=0.810, height=27, width=111)
        self.Label2.configure(background="#d9d9d9")
        self.Label2.configure(disabledforeground="#a3a3a3")
        self.Label2.configure(foreground="#000000")
        self.Label2.configure(text='''Закрывать:''')

        self.Scale2 = tk.Scale(self.Frame1, from_=0.0, to=50.0)
        self.Scale2.place(relx=0.58, rely=0.787, relwidth=0.147, relheight=0.0
                , height=47, bordermode='ignore')
        self.Scale2.configure(activebackground="#ececec")
        self.Scale2.configure(background="#d9d9d9")
        self.Scale2.configure(foreground="#000000")
        self.Scale2.configure(highlightbackground="#d9d9d9")
        self.Scale2.configure(highlightcolor="black")
        self.Scale2.configure(orient="horizontal")
        self.Scale2.configure(troughcolor="#d9d9d9")

        self.Label3 = tk.Label(self.Frame1)
        self.Label3.place(relx=0.487, rely=0.890, height=27, width=113)
        self.Label3.configure(background="#d9d9d9")
        self.Label3.configure(disabledforeground="#a3a3a3")
        self.Label3.configure(foreground="#000000")
        self.Label3.configure(text='''Эрозия:''')

        self.Scale3 = tk.Scale(self.Frame1, from_=0.0, to=50.0)
        self.Scale3.place(relx=0.58, rely=0.867, relwidth=0.147, relheight=0.0
                , height=47, bordermode='ignore')
        self.Scale3.configure(activebackground="#ececec")
        self.Scale3.configure(background="#d9d9d9")
        self.Scale3.configure(foreground="#000000")
        self.Scale3.configure(highlightbackground="#d9d9d9")
        self.Scale3.configure(highlightcolor="black")
        self.Scale3.configure(orient="horizontal")
        self.Scale3.configure(troughcolor="#d9d9d9")

        self.Label4 = tk.Label(self.Frame1)
        self.Label4.place(relx=0.021, rely=0.740, height=26, width=171)
        self.Label4.configure(background="#d9d9d9")
        self.Label4.configure(disabledforeground="#a3a3a3")
        self.Label4.configure(foreground="#000000")
        self.Label4.configure(text='''Диаметр опухоли :''')

        self.Label4_1 = tk.Label(self.Frame1)
        self.Label4_1.place(relx=0.021, rely=0.808, height=26, width=171)
        self.Label4_1.configure(activebackground="#f9f9f9")
        self.Label4_1.configure(activeforeground="black")
        self.Label4_1.configure(background="#d9d9d9")
        self.Label4_1.configure(disabledforeground="#a3a3a3")
        self.Label4_1.configure(foreground="#000000")
        self.Label4_1.configure(highlightbackground="#d9d9d9")
        self.Label4_1.configure(highlightcolor="black")
        self.Label4_1.configure(text='''2-d область опухоли :''')

        self.Label4_2 = tk.Label(self.Frame1)
        self.Label4_2.place(relx=0.03, rely=0.877, height=26, width=172)
        self.Label4_2.configure(activebackground="#f9f9f9")
        self.Label4_2.configure(activeforeground="black")
        self.Label4_2.configure(background="#d9d9d9")
        self.Label4_2.configure(disabledforeground="#a3a3a3")
        self.Label4_2.configure(foreground="#000000")
        self.Label4_2.configure(highlightbackground="#d9d9d9")
        self.Label4_2.configure(highlightcolor="black")
        self.Label4_2.configure(text='''Стадия опухоли:''')

        self.diameter = tk.Label(self.Frame1)
        self.diameter.place(relx=0.15, rely=0.740, height=27, width=382)
        self.diameter.configure(activebackground="#f9f9f9")
        self.diameter.configure(activeforeground="black")
        self.diameter.configure(background="#ffffff")
        self.diameter.configure(cursor="fleur")
        self.diameter.configure(disabledforeground="#a3a3a3")
        self.diameter.configure(foreground="#000000")
        self.diameter.configure(highlightbackground="#d9d9d9")
        self.diameter.configure(highlightcolor="black")

        self.area = tk.Label(self.Frame1)
        self.area.place(relx=0.15, rely=0.808, height=27, width=382)
        self.area.configure(activebackground="#f9f9f9")
        self.area.configure(activeforeground="black")
        self.area.configure(background="#ffffff")
        self.area.configure(cursor="fleur")
        self.area.configure(disabledforeground="#a3a3a3")
        self.area.configure(foreground="#000000")
        self.area.configure(highlightbackground="#d9d9d9")
        self.area.configure(highlightcolor="black")

        self.tumorstage = tk.Label(self.Frame1)
        self.tumorstage.place(relx=0.15, rely=0.877, height=27, width=382)
        self.tumorstage.configure(activebackground="#f9f9f9")
        self.tumorstage.configure(activeforeground="black")
        self.tumorstage.configure(background="#ffffff")
        self.tumorstage.configure(disabledforeground="#a3a3a3")
        self.tumorstage.configure(foreground="#000000")
        self.tumorstage.configure(highlightbackground="#d9d9d9")
        self.tumorstage.configure(highlightcolor="black")

	# -------------------------------------Image browser function----------------------------------------------------------
    def browse(self):
        img = Image.open(filedialog.askopenfilename())
        val = img.size
		# Actual size to resized image percentage calculation
        calc_area_percentage = ((477*404)/(val[0]*val[1]))*100

		# Resizing the image
        resized = img.resize((477,404), Image.Resampling.LANCZOS)

		# convert the image to ImageTk format
        imgtk = ImageTk.PhotoImage(image=resized)

		# storing image in the  browselabel
        self.browselabel.image = imgtk
        self.browselabel.configure(image=imgtk)

		# calling enhance_img function to enhance the image
        self.enhance_img(resized,calc_area_percentage)
        sys.stdout.flush()

	# -----------------------------------Image enhancement function-----------------------------------------------------------
    def enhance_img(self,img,area):
		# convert image to grayscale image
        resized_gray = cv2.cvtColor(np.float32(img), cv2.COLOR_BGR2GRAY)

		# apply thresholding
        thresh,img_bin = cv2.threshold(resized_gray,145,255,cv2.THRESH_BINARY)

		# getting opening, closing, erosion kernel values from the scale bars in the GUI
        val1 = self.Scale1.get()
        val2 = self.Scale2.get()
        val3 = self.Scale3.get()

		# IMPLEMENTING KERNELS
        kernel = np.ones((val1,val1), np.uint8)
        kernel2 = np.ones((val2,val2), np.uint8)

		# APPLYING MORPHOLOGICAL OPERATIONS TO THE IMAGE
        opening = cv2.morphologyEx(img_bin, cv2.MORPH_OPEN, kernel)
        closing = cv2.morphologyEx(opening, cv2.MORPH_CLOSE, kernel2)
        erosion = cv2.erode(closing,cv2.getStructuringElement(cv2.MORPH_ELLIPSE,(val3,val3)))

		# CROP IMAGE TO EXTRACT THE TUMOR PART
        crop_img = erosion[50:400, 60:477]

		# convert the image to ImageTk format
        image1 = Image.fromarray(crop_img)
        imgtk = ImageTk.PhotoImage(image=image1)

		# storing image in the  enhanceimage label
        self.enhanceimage.image = imgtk
        self.enhanceimage.configure(image=imgtk)

		# writing the enhanced image to the working directory
        cv2.imwrite("img.png", crop_img)

		# calling the image segmenting function
        self.segment_img(area)
        sys.stdout.flush()



	#----------------------------------------Image segmentation function----------------------------------------------------------
    def segment_img(self,area):
        thresh = cv2.imread('img.png',0)
		# check if the image is black without any detected tumors in white objects

        if np.mean(thresh) == 0:
			# convert the image to ImageTk format
            image1 = Image.fromarray(thresh)
            imgtk = ImageTk.PhotoImage(image=image1)

			# storing image in the  segImg label
            self.segImg.image = imgtk
            self.segImg.configure(image=imgtk)

			# return messages to the following labels
            self.diameter.configure(text="Отека не обнаружено.")
            self.area.configure(text="Отека не обнаружено.")
            self.tumorstage.configure(text="Отека не обнаружено.")
        else:
			# find contours
            contours,hierarchy = cv2.findContours(thresh,2,1)  #include another variable before 'contuors' if you get an error saying "ValueError: not enough values to unpack (expected 3, got 2)"
			# convert image to grayscale image
            thresh = cv2.cvtColor(thresh, cv2.COLOR_GRAY2RGB)
            cnt = contours
			# draw contours on the image
            cv2.drawContours(thresh, cnt, -1, (36, 255, 12), 2)

            arr = [0 for i in cnt]

			# finding the radius of the segmented objects
            for i in range (len(cnt)):
                (x,y),radius = cv2.minEnclosingCircle(cnt[i])
                center = (int(x),int(y))
                radius = int(radius)
                cv2.circle(thresh,center,radius,(0,255,0),2)
			# populating the diameters of the objects to arr
                arr[i] = radius*0.26458333

            # convert the image to ImageTk format
            image1 = Image.fromarray(thresh)
            imgtk = ImageTk.PhotoImage(image=image1)
	    # storing image in the  segImg label
            self.segImg.image = imgtk
            self.segImg.configure(image=imgtk)
	    # sorting the array in descending order
            arr.sort(reverse=True)
	    # find the area of the biggest tumor in the image
            area_circle = math.pi * (arr[0]*arr[0])
	    # finding the area for the actual sized image
            actual_circle_area = area_circle*(100/area)
	    # calculating the diameter from the actual circle area
            diameter_result1 = round(math.sqrt((actual_circle_area/math.pi))*2, ndigits=2)
            diameter_result2 = "(Реальное изображение) Диаметр самой большой опухоли " + str(diameter_result1)  + "mm"

	    # storing the diameter in the diameter label
            self.diameter.configure(text=diameter_result2)
            result = round(actual_circle_area, ndigits=2)
            resultant_area = "(Реальное изображение) Самая большая площадь опухоли " + str(result) + "mm"

			# storing the area of the tumor in the area label
            self.area.configure(text=resultant_area)
			# calling the tumor_st function
            self.tumor_st(float(diameter_result1/10))
        sys.stdout.flush()

	# --------------Tumor stage identifying function--------------
    def tumor_st(self,diameter):
        if diameter == 0:
            self.tumorstage.configure(text="Стадия опухоли T0")

        elif diameter > 0 and diameter <= 2:
            if diameter > 0 and diameter <= 0.1:
                self.tumorstage.configure(text="Стадия опухоли T1")
            elif diameter > 0.1 and diameter <= 0.5:
                self.tumorstage.configure(text="Стадия опухоли T1a")
            elif diameter > 0.5 and diameter <= 1:
                self.tumorstage.configure(text="Стадия опухоли T1b")
            else:
                self.tumorstage.configure(text="Стадия опухоли T1c")

        elif (diameter > 2) and (diameter <= 5):
            self.tumorstage.configure(text="Стадия опухоли T2")
        elif diameter > 5:
            self.tumorstage.configure(text="Стадия опухоли T3")
        else:
            self.tumorstage.configure(text="Стадия опухоли T4")
        sys.stdout.flush()

if __name__ == '__main__':
    vp_start_gui()




