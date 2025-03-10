import RPi.GPIO as GPIO
import time
import picamera
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

# GPIO Pin configuration
DOORBELL_BUTTON = 17  # GPIO pin for the button
GPIO.setmode(GPIO.BCM)
GPIO.setup(DOORBELL_BUTTON, GPIO.IN, pull_up_down=GPIO.PUD_UP)

# Camera setup
camera = picamera.PiCamera()

# Email Configuration
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
EMAIL_ADDRESS = "your_email@gmail.com"  # Replace with your email
EMAIL_PASSWORD = "your_password"  # Replace with your password
RECIPIENT_EMAIL = "recipient_email@gmail.com"  # Replace with recipient's email

# Function to capture image
def capture_image():
    image_path = "doorbell_image.jpg"
    camera.capture(image_path)
    print("Image captured")
    return image_path

# Function to send email with image attachment
def send_email(image_path):
    msg = MIMEMultipart()
    msg['From'] = EMAIL_ADDRESS
    msg['To'] = RECIPIENT_EMAIL
    msg['Subject'] = "Smart Doorbell Alert"
    
    with open(image_path, "rb") as attachment:
        part = MIMEBase("application", "octet-stream")
        part.set_payload(attachment.read())
        encoders.encode_base64(part)
        part.add_header("Content-Disposition", f"attachment; filename={image_path}")
        msg.attach(part)
    
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
        server.sendmail(EMAIL_ADDRESS, RECIPIENT_EMAIL, msg.as_string())
        print("Email sent successfully")

# Main function to monitor doorbell press
if __name__ == "__main__":
    print("Smart Doorbell Ready... Press the button to capture an image")
    try:
        while True:
            if GPIO.input(DOORBELL_BUTTON) == GPIO.LOW:
                print("Doorbell pressed!")
                image = capture_image()
                send_email(image)
                time.sleep(5)  # Debounce delay
    except KeyboardInterrupt:
        print("Exiting...")
        GPIO.cleanup()

