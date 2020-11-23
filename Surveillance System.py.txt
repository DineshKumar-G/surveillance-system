'''including SIMPLE MAIL TRANSFER PROTOCOL'''
import smtplib
'''including library for PIR SENSOR'''
from gpiozero import MotionSensor
'''including library for PI CAMERA'''
from picamera import PiCamera
'''including library to handle TIMING EVENTS'''
import time
from subprocess import call
import os
'''including MAIL TRANSFER PROTOCOL LIBRARY'''
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders
from email.mime.image import MIMEImage
'''INITIATING THE PI CAMERA'''
camera = PiCamera()
camera.resolution = (640, 480)
camera.close
pir = MotionSensor(4)
count=11
while True:   
    '''filename = "intruder"+str(count)+".h264"'''
    '''filename = str(str(datetime.now())+".h264")'''
    filename = str(time.strftime("%Y-%m-%d.%H:%M"))+".h264"
    '''checks for MOTION'''
    pir.wait_for_motion()
    print("Motion Detected")
    '''sends mail alert'''
    fromaddr = "raspb027@gmail.com"
    toaddr = "agasthya30@gmail.com"
    toaddr = "dineshwinchester@gmail.com"
    mail = MIMEMultipart()
    mail['From'] = fromaddr
    mail['To'] = toaddr
    mail['Subject'] = "ALERT!!"
    body = "Motion Detected"
    mail.attach(MIMEText(body,'plain'))
    server = smtplib.SMTP('smtp.gmail.com',587)
    server.starttls()
    server.login(fromaddr,"dinesh@gmail.com")
    text = mail.as_string()
    server.sendmail(fromaddr, toaddr, text)
    server.quit()
    print("Email Sent")
    count = count+1
    '''Filename of the video is given in terms of timestamp of the video recording'''
    camera.start_recording(filename)
    '''CAMERA ORIENTATION'''
    camera.rotation=0
    print("Start Recording - ",count)
    '''once the motion is detected it records of the next 10 seconds'''
    time.sleep(5)
    '''STOPS RECORDING'''
    camera.stop_recording()
    print("Recording stopped")
    '''to convert the h264 video to mp4 format for viewing the video in mobile and laptops'''
    mp4filename = filename+".mp4"
    comm1 = "avconv -r 25 -i /home/pi/Desktop/C/"+filename+" -vcodec copy /home/pi/Desktop/C/"+mp4filename
    os.system(comm1)
    print("Uploading "+mp4filename+ " to Dropbox")
    comm = "/home/pi/Desktop/C/Dropbox-Uploader/dropbox_uploader.sh upload "+mp4filename+" /"
    call([comm],shell=True)
    print("Upload done :)")
