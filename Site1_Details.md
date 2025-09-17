# Project Technical & Architectural Breakdown

### **Live Application:** [https://vedicastro101.com](https://vedicastro101.com)

---

## **Technical Architecture**

The application follows a decoupled, client-server architecture. The frontend is a Single-Page Application (SPA) built with React, while the backend is a RESTful API service built with Python and Flask. This separation of concerns allows for independent development, scaling, and maintenance of the client and server.

#### **High-Level Architectural Flow:**

1.  **User Interaction:** The user accesses the website hosted on **Cloudflare Pages**. Cloudflare provides global CDN distribution, security, and SSL.
2.  **Frontend Rendering:** The React application is served to the user's browser. All UI logic, state management, and routing are handled client-side.
3.  **API Communication:** When the user submits the birth chart form, the React frontend makes an asynchronous `POST` request to a dedicated backend API endpoint (`api.vedicastro101.com`).
4.  **Backend Processing:** The API request is routed through **Cloudflare's proxy** (providing WAF, rate limiting, and DDoS protection) to a **Hetzner VPS** running Ubuntu Server.
5.  **Reverse Proxy:** **Nginx** acts as a reverse proxy, receiving the HTTPS request, terminating SSL, and forwarding it to the **Gunicorn** application server.
6.  **Application Logic:** Gunicorn manages the **Flask** application. The Flask API processes the input data, performs complex astrological calculations using the **`swisseph`** library, and generates custom SVG charts using **`svgwrite`**.
7.  **Response:** The API returns a comprehensive JSON object containing all calculated data and the SVG chart strings, which the React frontend then parses and renders dynamically for the user.

---

## **Technology Stack**

### **Frontend (Client-Side)**

*   **JavaScript (ES6+):** The primary language for all frontend logic.
*   **React (v18+):** Used for building a component-based, declarative UI. The SPA architecture ensures a fast and fluid user experience.
*   **React Router:** Implemented for client-side routing and passing state between routes to preserve form data.
*   **HTML5 & CSS3:** Used for structuring content and styling. The application features a modern, responsive design with a dark theme, utilizing Flexbox, Grid, and media queries to adapt across all devices.
*   **Fuse.js:** A lightweight fuzzy-search library implemented to create an efficient city search feature.
*   **`date-and-time`:** An NPM package used for robust client-side date validation to ensure data integrity.

### **Backend (Server-Side)**

*   **Python 3:** The core language for the backend, chosen for its powerful libraries for scientific computing.
*   **Flask:** A lightweight WSGI web application framework used to build the REST API.
*   **Gunicorn:** A production-grade WSGI HTTP server used to run and manage the Flask application.
*   **`swisseph`:** A high-precision astronomical ephemeris library that forms the backbone of the calculation engine.
*   **`pytz` & `timezonefinder`:** Used for accurate timezone conversions, translating local birth time into UTC.
*   **`svgwrite`:** A Python library used to programmatically generate scalable and dynamic North Indian SVG birth charts.
*   **`python-dateutil`:** Utilized for performing complex and precise date arithmetic for calculating Vimshottari Dasha periods.

### **Infrastructure & Deployment (DevOps)**

*   **Hosting:**
    *   **Frontend:** Deployed on **Cloudflare Pages**, providing CI/CD, global CDN, and automatic SSL.
    *   **Backend:** Hosted on a **Hetzner Virtual Private Server (VPS)**.
*   **Operating System:** **Ubuntu Server**.
*   **Web Server / Reverse Proxy:** **Nginx** is configured as a reverse proxy for SSL termination, request routing, and implementing security measures.
*   **Process Management:** **Systemd** is used to manage the Gunicorn service, ensuring the API runs persistently as a daemon.
*   **Security:**
    *   **SSL/TLS:** End-to-end HTTPS encryption enforced using **Let's Encrypt (Certbot)** and Cloudflare's "Full (Strict)" SSL mode.
    *   **Firewall:** **UFW (Uncomplicated Firewall)** is configured on the server to restrict traffic to essential ports.
    *   **Cloudflare:** Provides a WAF, DDoS mitigation, and domain-specific rate limiting rules.
    *   **SSH Hardening:** The server is secured with key-based authentication, with password authentication and root login disabled.
*   **CORS:** The Flask backend has a strict Cross-Origin Resource Sharing (CORS) policy, configured to accept requests exclusively from the frontend domain.

---

## **Core Technical Challenges & Solutions**

1.  #### **Dynamic SVG Chart Generation**
    *   **Challenge:** Creating visually accurate and aesthetically pleasing North Indian charts entirely on the backend without relying on a third-party charting library.
    *   **Solution:** I developed a custom function using the `svgwrite` library. This involved meticulously plotting the geometric coordinates for each of the 12 houses and defining precise placement logic for sign numbers. A sophisticated algorithm was created to handle the dynamic placement of multiple planets within a single house, calculating vertical offsets to ensure even distribution and prevent overlap.

2.  #### **Precision Astrological & Date Calculations**
    *   **Challenge:** Vedic Astrology requires high-precision calculations. The Vimshottari Dasha system, in particular, involves projecting planetary cycles over 120 years, where minor inaccuracies can lead to significant errors.
    *   **Solution:** I leveraged the `swisseph` library for astronomical precision. For Dasha calculations, I implemented custom helper functions using Python's `datetime` and `dateutil.relativedelta` to handle fractional days and varying year-length definitions (Tropical, Sidereal, Savana), ensuring accuracy over long time spans.

3.  #### **State Management & User Experience**
    *   **Challenge:** A key usability requirement was to preserve the user's form data across navigation to prevent the need for re-entry, which would create a poor user experience.
    *   **Solution:** I implemented a two-pronged data persistence strategy. Form state is passed to the results page via **React Router's `location.state`** and is simultaneously saved to the browser's **`sessionStorage`**. This ensures that if the user navigates away and later returns, their data is seamlessly reloaded into the form.

4.  #### **Secure and Production-Ready API Deployment**
    *   **Challenge:** Deploying a Python web application securely and efficiently, moving beyond the simple development server.
    *   **Solution:** I designed a multi-layered, production-grade deployment architecture. The Flask application is served by a robust **Gunicorn** server, managed by a **Systemd** service for reliability. **Nginx** acts as a high-performance reverse proxy, handling SSL and providing security. The entire infrastructure is further shielded by **Cloudflare**, which provides CDN caching, rate limiting, and protection against common web vulnerabilities.
