# MY_PROJECTS

import cv2
import numpy as np
import pyautogui

# Constants
VIDEO_WIDTH, VIDEO_HEIGHT = 1280, 720  # Resolution of the camera (720p)
SCREEN_WIDTH, SCREEN_HEIGHT = 1920, 1080  # Resolution of the screen
MONITOR_SIZE_INCH = 19  # Size of the monitor in inches
CURSOR_SPEED = 10  # Cursor movement speed

# Function to move cursor based on circle position
def move_cursor(circle_x, circle_y):
    # Convert circle position to screen coordinates
    screen_x = int(circle_x * SCREEN_WIDTH / VIDEO_WIDTH)
    screen_y = int(circle_y * SCREEN_HEIGHT / VIDEO_HEIGHT)
    
    # Convert screen coordinates to monitor inches
    monitor_x = screen_x * MONITOR_SIZE_INCH / SCREEN_WIDTH
    monitor_y = screen_y * MONITOR_SIZE_INCH / SCREEN_HEIGHT
    
    # Move cursor
    pyautogui.moveTo(monitor_x, monitor_y, duration=CURSOR_SPEED/1000)

# Function to detect red circles in the video stream
def detect_red_circles(frame):
    # Convert frame to HSV color space
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    
    # Define lower and upper bounds for red color
    lower_red = np.array([0, 100, 100])
    upper_red = np.array([10, 255, 255])
    
    # Threshold the HSV image to get only red colors
    mask = cv2.inRange(hsv, lower_red, upper_red)
    
    # Find contours in the mask
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    # Iterate through contours and detect circles
    for contour in contours:
        area = cv2.contourArea(contour)
        if area > 100:  # Adjust this threshold based on your setup
            # Fit a circle to the contour
            ((x, y), radius) = cv2.minEnclosingCircle(contour)
            
            # Convert radius to inches
            radius_inches = radius * MONITOR_SIZE_INCH / SCREEN_WIDTH
            
            # If circle radius is approximately 2 cm (0.79 inches) and color is red
            if abs(radius_inches - 0.79) < 0.1:  # Adjust tolerance based on your setup
                # Draw circle and centroid on the frame
                cv2.circle(frame, (int(x), int(y)), int(radius), (0, 255, 255), 2)
                cv2.circle(frame, (int(x), int(y)), 5, (0, 0, 255), -1)
                
                # Move cursor based on circle position
                move_cursor(x, y)

# Main function
def main():
    # Open camera
    cap = cv2.VideoCapture(0)
    cap.set(3, VIDEO_WIDTH)
    cap.set(4, VIDEO_HEIGHT)

    while True:
        # Capture frame-by-frame
        ret, frame = cap.read()
        if not ret:
            break
        
        # Detect red circles
        detect_red_circles(frame)
        
        # Display the resulting frame
        cv2.imshow('Frame', frame)
        
        # Break the loop if 'q' is pressed
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    # Release the capture
    cap.release()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    main()
