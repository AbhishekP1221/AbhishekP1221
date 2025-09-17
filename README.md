# Hi, I'm Abhishek 👋

I'm a software engineer who enjoys building interesting applications and learning new technologies.

## 🧠 Skills
- Languages: Python, Java, SQL, C++, JavaScript
- Frameworks: React, Flask, Node.js, Express.js
- Tools: Git, Linux, VS Code, IntelliJ,

I have built and managage two websites which I'm describing below.

Project: Full-Stack Vedic Astrology Birth Chart Generator
Live Application: https://vedicastro101.com/

Project Overview
This is a full-stack web application designed to generate accurate and detailed Vedic Astrology (Jyotish) birth charts. The application provides a dynamic, responsive user interface for data input and presents complex astrological data in a clean, digestible, and mobile-friendly format. The backend is a robust REST API built to handle precise astronomical calculations and generate custom SVG charts on the fly. The entire project is architected for security, performance, and scalability, demonstrating a comprehensive understanding of the full software development lifecycle.
The primary goal of this project was to create a highly flexible and user-friendly tool that caters to both astrology beginners and advanced practitioners by offering a wide array of customizable options (e.g., Ayanamsa, Karaka schemes) not commonly found in free online calculators.

Technical Architecture
The application follows a decoupled, client-server architecture. The frontend is a Single-Page Application (SPA) built with React, while the backend is a RESTful API service built with Python and Flask. This separation of concerns allows for independent development, scaling, and maintenance of the client and server.

High-Level Architectural Flow:
User Interaction: The user accesses the website hosted on Cloudflare Pages. Cloudflare provides global CDN distribution, security, and SSL.
Frontend Rendering: The React application is served to the user's browser. All UI logic, state management, and routing are handled client-side.
API Communication: When the user submits the birth chart form, the React frontend makes an asynchronous POST request to a dedicated backend API endpoint (api.vedicastro101.com).
Backend Processing: The API request is routed through Cloudflare's proxy (providing WAF, rate limiting, and DDoS protection) to a Hetzner VPS running Ubuntu Server.
Reverse Proxy & Load Balancing: Nginx acts as a reverse proxy, receiving the HTTPS request, terminating SSL, and forwarding it to the Gunicorn application server.
Application Logic: Gunicorn manages multiple worker processes for the Flask application. The Flask API processes the input data, performs complex astrological calculations using the swisseph library, and generates custom SVG charts using svgwrite.
Response: The API returns a comprehensive JSON object containing all calculated data and the SVG chart strings, which the React frontend then parses and renders dynamically for the user.

Technology Stack
Frontend (Client-Side)
JavaScript (ES6+): The primary language for all frontend logic.
React (v18+): Used for building a component-based, declarative UI. The SPA architecture ensures a fast and fluid user experience without page reloads.
React Router: Implemented for client-side routing, enabling navigation between different "pages" (e.g., Home, Results, About) within the SPA. Also used to pass state between routes to preserve form data.
HTML5 & CSS3: Used for structuring content and styling. The application features a modern, responsive design with a dark theme, utilizing Flexbox, Grid, and media queries to adapt seamlessly across desktops, tablets, and mobile devices.
Fuse.js: A lightweight fuzzy-search library implemented to create a smart and efficient city search feature, allowing users to easily find their birth location from a large JSON dataset.
date-and-time: An NPM package used for robust client-side date validation to ensure data integrity before making an API call.

Backend (Server-Side)
Python 3: The core language for the backend, chosen for its powerful libraries for scientific and astronomical computing.
Flask: A lightweight WSGI web application framework used to build the REST API. Its simplicity and flexibility were ideal for creating a dedicated microservice.
Gunicorn: A production-grade WSGI HTTP server used to run and manage the Flask application. It provides robust process management with multiple workers.
swisseph: A high-precision astronomical ephemeris library that forms the backbone of the calculation engine. It is used to determine the precise longitudinal positions of celestial bodies.
pytz & timezonefinder: Used for accurate timezone conversions, translating the user's local birth time into UTC for ephemeris calculations.
svgwrite: A Python library used to programmatically generate scalable, dynamic, and custom-styled North Indian SVG birth charts based on the calculated planetary data.
python-dateutil: Utilized for performing complex and precise date arithmetic, which is critical for calculating the multi-decade Vimshottari Dasha periods.

Infrastructure & Deployment (DevOps)
Hosting:
Frontend: Deployed on Cloudflare Pages, which provides CI/CD, global CDN distribution, automatic SSL, and tight integration with other Cloudflare services.
Backend: Hosted on a Hetzner Virtual Private Server (VPS), providing a cost-effective and powerful Linux environment.
Operating System: Ubuntu Server, a stable and widely-supported Linux distribution.
Web Server / Reverse Proxy: Nginx is configured as a reverse proxy to sit in front of the Gunicorn application server. Its roles include SSL termination, request routing, serving static files (if any), and implementing security measures like rate limiting and blocking non-browser clients.
Process Management: Systemd is used to manage the Gunicorn service, ensuring the API runs persistently as a daemon and restarts automatically on failure or server reboot.
Security:
SSL/TLS: End-to-end HTTPS encryption is enforced using a certificate from Let's Encrypt (Certbot) on the Nginx server and Cloudflare's "Full (Strict)" SSL mode.
Firewall: UFW (Uncomplicated Firewall) is configured on the server to restrict traffic to essential ports (SSH, HTTP, HTTPS).
Cloudflare: Provides an additional security layer, including a Web Application Firewall (WAF), DDoS mitigation, and domain-specific rate limiting rules.
SSH Hardening: The server is secured with key-based authentication, with password authentication and root login disabled.
CORS: The Flask backend has a strict Cross-Origin Resource Sharing (CORS) policy, configured to accept requests exclusively from the frontend domain (vedicastro101.com).

Core Technical Challenges & Solutions
Dynamic SVG Chart Generation:
Challenge: Creating visually accurate and aesthetically pleasing North Indian charts entirely on the backend without using a charting library.
Solution: I developed a custom function using the svgwrite library. This involved meticulously plotting the geometric coordinates for each of the 12 houses and defining precise placement logic for sign numbers. A sophisticated algorithm was created to handle the dynamic placement of multiple planets within a single house, calculating vertical offsets to ensure even distribution and prevent overlap, with priority-based positioning to avoid overflow in crowded chart houses.
Precision Astrological & Date Calculations:
Challenge: Vedic Astrology requires high-precision calculations. The Vimshottari Dasha system, in particular, involves projecting planetary cycles over 120 years, where minor inaccuracies in year length or date arithmetic can lead to significant errors.
Solution: I leveraged the swisseph library for astronomical precision. For Dasha calculations, I implemented custom helper functions using Python's datetime and dateutil.relativedelta modules. These functions handle fractional days and varying year-length definitions (Tropical, Sidereal, Savana), ensuring that the dasha period calculations are accurate to the day over very long time spans.
State Management & User Experience:
Challenge: A key usability requirement was to preserve the user's form data, even after they navigate to the results page or other parts of the website. Forcing users to re-enter complex birth data repeatedly would create a poor user experience.
Solution: I implemented a two-pronged data persistence strategy. First, on form submission, the form state is passed to the results page via React Router's location.state. Second, the form data is simultaneously saved to the browser's sessionStorage. This ensures that if the user navigates away from the results page and later returns to the home page, their data is seamlessly reloaded into the form, making it easy to tweak and regenerate charts.
Secure and Production-Ready API Deployment:
Challenge: Deploying a Python web application securely and efficiently, moving beyond the simple development server.
Solution: I designed a multi-layered, production-grade deployment architecture. The Flask application is served by a robust Gunicorn server with multiple workers, managed by a Systemd service for reliability. Nginx acts as a high-performance reverse proxy, handling SSL and providing an initial layer of security by filtering requests. The entire infrastructure is further shielded by Cloudflare, which provides CDN caching, rate limiting, and protection against common web vulnerabilities, ensuring the API is secure, performant, and reliable.
