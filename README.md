# Dual-Modality Attendance System

A web-based attendance system that verifies students through **four independent checks** before logging attendance: face recognition, a rotating QR code, GPS geofencing, and a live anti-spoofing challenge. Built as a Final Year Project at Multimedia University (MMU).

## Why

Traditional attendance methods — sign-in sheets, RFID cards, and static QR codes — can be faked, shared, or submitted from anywhere. This system requires a student to prove **who they are** (face match), **where they are** (GPS), **a currently valid QR token** (anti-forwarding), and **that they are physically present right now** (liveness check) — all from a single smartphone, with no extra hardware.

## Features

- **Face Recognition** — MTCNN for face detection and FaceNet (512-D embeddings) for identity matching, with a 0.50 cosine-similarity threshold
- **Dynamic QR Code** — lecturer's QR token rotates every 10 seconds (MD5-based), with a 30-second scan window to prevent screenshot forwarding
- **GPS Geofencing** — Haversine distance check between the student's location and a configurable classroom centre point (100–500 m radius)
- **Liveness Challenge** — student completes 2 of 4 randomised actions (blink, mouth movement, head-turn) via MediaPipe to resist photo/video replay attacks
- **Role-Based Dashboards** — separate student, lecturer, and admin views with real-time attendance records
- **Duplicate Rejection** — prevents the same student from being logged twice for one session

## Tech Stack

- **Frontend:** HTML, CSS, JavaScript, Geolocation API, camera capture
- **Backend:** PHP (Apache via XAMPP)
- **Face Recognition Service:** Python (Flask API), MTCNN, FaceNet (keras-facenet), MediaPipe, OpenCV
- **Database:** MySQL
- **QR Code Generation:** [phpqrcode](https://sourceforge.net/projects/phpqrcode/)

## System Architecture

The system runs on a three-tier architecture:

- **Presentation Tier** — mobile browser handles the QR scanner, camera capture, geolocation, and liveness challenge UI
- **Business Logic Tier** — PHP/Apache validates QR tokens, timetable/time windows, and GPS distance, and calls the Python Flask API for face matching
- **Data Tier** — MySQL stores users, students, lecturers, subjects, timetables, classroom locations, and attendance records; a `face_database.json` file stores face embeddings

## Setup

### Prerequisites
- XAMPP (Apache + MySQL + PHP 8)
- Python 3.x
- A webcam-enabled device with a modern mobile browser

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/<repo-name>.git
```

### 2. Set up the database
Create a MySQL database named `attendance_fyp` and import `attendance_fyp.sql`, which contains the schema for `users`, `students`, `lecturers`, `subjects`, `student_subject`, `timetable`, `classroom_location`, and `attendance`.

```bash
mysql -u root -p attendance_fyp < attendance_fyp.sql
```

### 3. Configure the database connection
Update `connection.php` with your MySQL credentials if different from the defaults:
```php
$host = "localhost";
$user = "root";
$pass = "";
$db   = "attendance_fyp";
$port = 3306;
```

### 4. Install Python dependencies
```bash
pip install opencv-python mediapipe mysql-connector-python mtcnn keras-facenet flask numpy
```

### 5. Run the Flask face recognition API
```bash
python face_api.py
```
The API runs on `http://127.0.0.1:5001`.

### 6. Run the PHP application
Place the project folder inside your XAMPP `htdocs` directory, start Apache and MySQL, then access the app via `http://localhost/<project-folder>/login.php` (or via an ngrok HTTPS tunnel for camera/geolocation access on mobile browsers, since these APIs require a secure context).

## Usage Flow

1. Lecturer starts a session; a QR code is generated and refreshes every 10 seconds
2. Student opens the attendance page and scans the QR code within the 30-second window
3. Student completes 2 of 4 random liveness actions
4. Student's camera and GPS location are captured together
5. The system verifies all four layers — face match, QR validity, GPS distance, and liveness — before logging attendance with a face-accuracy score

## Known Limitations

- Face-match accuracy peaks at 89.0% under normal lighting but drops to 65–74% with glasses or poor lighting
- Indoor GPS readings are less accurate than outdoor readings (signal attenuation/reflection through walls and roofs); classroom centre coordinates should be entered manually via Google Maps rather than captured indoors
- No server-side validation currently prevents a modified/jailbroken device from spoofing browser-reported GPS coordinates
- Tested primarily on Android; Safari/iOS location permission handling needs further work

## Future Work

- Acoustic liveness sensing (AFace-style)
- Wi-Fi access-point cross-validation for stronger location proof
- Repeated GPS readings to reduce single-reading anomalies
- Resolving Safari/iOS geolocation permission issues

## Author

**Cheng Theng Ler**
Faculty of Information Science and Technology, Multimedia University
Supervised by Ms. Sharifah Noor Masidayu binti Sayed Ismail

## License

This project was developed for academic purposes as part of a Final Year Project.
