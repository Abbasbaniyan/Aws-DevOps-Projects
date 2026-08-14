# Student Registration Web Application

## Project

This project is a Java web application deployed on Apache Tomcat. It is connected with AWS RDS MySQL database. Nginx is used as a reverse proxy to access the application.

## AWS Services Used

- EC2
- RDS (MySQL)
- Security Groups
- VPC

## Software Used

- Amazon Linux 2023
- Java 17
- Apache Tomcat 9
- Nginx
- MySQL
- JDBC

## Project Flow

1. Launch EC2 instance.
2. Install Java and Tomcat.
3. Deploy Student Registration WAR file.
4. Create MySQL database in AWS RDS.
5. Connect Tomcat with RDS using JDBC.
6. Configure Nginx as reverse proxy.
7. Open application in browser.
8. Register student details.
9. Data gets stored in RDS database.

## Screenshots

### 1. Java Installed

<img width="1917" height="312" alt="Screenshot 2026-08-07 192215" src="https://github.com/user-attachments/assets/ab27c4fc-a52e-4fe6-8b51-72c47f0a6cca" />


---

### 2. Apache Tomcat Running

<img width="1900" height="977" alt="Screenshot 2026-08-07 192416" src="https://github.com/user-attachments/assets/7f87492f-f831-4a76-8164-ecb829231665" />


---

### 3. Student Registration Form

<img width="1912" height="977" alt="Screenshot 2026-08-07 193029" src="https://github.com/user-attachments/assets/13081775-0957-447d-8f0d-b66a60e5d46f" />

---

### 4. Database Connected

<img width="1911" height="955" alt="Screenshot 2026-08-07 194345" src="https://github.com/user-attachments/assets/771e698c-4cb9-40ba-8128-6300e76f1210" />


--

### 5. Student List

<img width="1506" height="636" alt="Screenshot 2026-08-07 200941" src="https://github.com/user-attachments/assets/fc6a3675-2b20-4235-8869-58a4e1e17dfa" />


---

### 7. Proxy Server (Nginx)

<img width="1902" height="890" alt="Screenshot 2026-08-07 202304" src="https://github.com/user-attachments/assets/c6da9976-d0e8-420a-950d-e189066df711" />


---

### 8. Application Access Through Proxy Server

<img width="1915" height="481" alt="Screenshot 2026-08-07 202937" src="https://github.com/user-attachments/assets/d443312c-60e4-4418-92eb-02b8a9e05f5f" />


## Result

The Student Registration application was successfully deployed on Apache Tomcat. The application is connected with AWS RDS MySQL database through JDBC. Nginx reverse proxy is also configured successfully. Student records are stored and displayed from the database.
