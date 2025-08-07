import cv2
import mediapipe as mp
import pyttsx3

mp_hands = mp.solutions.hands
hands = mp_hands.Hands(max_num_hands=1)
mp_draw = mp.solutions.drawing_utils

engine = pyttsx3.init()

# Try different indices if this doesn't work (0, 1, 2)
cap = cv2.VideoCapture(0)

if not cap.isOpened():
    print("❌ Could not open camera")
    exit()
else:
    print("✅ Camera opened successfully")

said_hello = False

while True:
    success, img = cap.read()
    if not success:
        print("❌ Failed to read frame from camera")
        break

    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(img_rgb)

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mp_draw.draw_landmarks(img, hand_landmarks, mp_hands.HAND_CONNECTIONS)

            thumb_tip = hand_landmarks.landmark[4]
            index_tip = hand_landmarks.landmark[8]

            dist = abs(thumb_tip.x - index_tip.x)
            if dist > 0.1 and not said_hello:
                engine.say("Hello")
                engine.runAndWait()
                said_hello = True
    else:
        said_hello = False

    cv2.imshow("Sign Language to Voice", img)
    if cv2.waitKey(1) & 0xFF == 27:
        break

cap.release()
cv2.destroyAllWindows()
