import cv2
import mediapipe as mp
import pyautogui
import time

# Initialize webcam and Mediapipe Hand Tracking
webcam = cv2.VideoCapture(0)
my_hands = mp.solutions.hands.Hands()
drawing_utils = mp.solutions.drawing_utils

# Variables to store last known positions
x1, y1, x2, y2 = None, None, None, None
prev_x1, prev_y1, prev_x2, prev_y2 = None, None, None, None
last_update_time = time.time()

while True:
    _, image = webcam.read()
    image = cv2.flip(image, 1)  # Flip to match real movement
    frame_height, frame_width, _ = image.shape
    rgb_image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    output = my_hands.process(rgb_image)
    hands = output.multi_hand_landmarks

    hand_detected = False  # Flag to check if hand is visible

    if hands:
        for hand in hands:
            drawing_utils.draw_landmarks(image, hand)
            landmarks = hand.landmark

            for id, landmark in enumerate(landmarks):
                x = int(landmark.x * frame_width)
                y = int(landmark.y * frame_height)

                if id == 8:  # Index finger tip
                    cv2.circle(image, (x, y), 8, (0, 255, 255), 3)
                    x1, y1 = x, y
                    hand_detected = True

                if id == 4:  # Thumb tip
                    cv2.circle(image, (x, y), 8, (0, 0, 255), 3)
                    x2, y2 = x, y
                    hand_detected = True

    # If hand is lost, stop volume control immediately
    if not hand_detected:
        x1 = y1 = x2 = y2 = None
        prev_x1 = prev_y1 = prev_x2 = prev_y2 = None  # Reset previous positions
        continue  # Skip to the next frame

    # Check if the hand is moving (Compare with previous position)
    if (
        prev_x1 is not None
        and prev_y1 is not None
        and prev_x2 is not None
        and prev_y2 is not None
    ):
        movement = abs(x1 - prev_x1) + abs(y1 - prev_y1) + abs(x2 - prev_x2) + abs(y2 - prev_y2)

        if movement < 10:  # If movement is very small, stop volume control
            continue  # Skip volume adjustment

    # Store the current position as previous for next frame
    prev_x1, prev_y1, prev_x2, prev_y2 = x1, y1, x2, y2

    # Calculate distance & adjust volume only if fingers are tracked
    if x1 is not None and x2 is not None:
        dist = ((x2 - x1) ** 2 + (y2 - y1) ** 2) ** 0.5 // 4
        cv2.line(image, (x1, y1), (x2, y2), (0, 255, 0), 5)

        # Adjust volume only if the hand is moving
        if dist > 50:
            pyautogui.press("volumeup")
        elif dist < 30:
            pyautogui.press("volumedown")

    # Display image
    cv2.imshow("Hand Volume Control", image)

    # Exit on 'Esc' keys
    key = cv2.waitKey(10)
    if key == 27:
        break

webcam.release()
cv2.destroyAllWindows()
