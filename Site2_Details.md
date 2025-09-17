# Project Technical & Architectural Breakdown

### **Live Application:** [https://discoversupps.com](https://discoversupps.com)

---

## **Technical Architecture**

The application is built on a modern, decoupled, three-tier architecture. This design separates the presentation layer (React frontend), the application logic layer (Node.js API), and the data layer (MongoDB), allowing for enhanced security, scalability, and maintainability.

#### **High-Level Architectural Flow:**

1.  **User Interaction:** The user visits **discoversupps.com**, which is hosted on **Cloudflare Pages**. Cloudflare provides a global CDN for fast delivery, automated CI/CD from GitHub, and robust security features.
2.  **Frontend Rendering:** The React 19 Single-Page Application (SPA) is served to the user's browser. All UI logic, filter state management, and routing are handled client-side.
3.  **API Communication:** When the user initiates a search, the React frontend constructs a JSON object from the selected filters and sends an asynchronous `POST` request to the dedicated backend API endpoint (`api.discoversupps.com`).
4.  **Backend Processing:** The API request is routed through **Cloudflare's proxy** (providing WAF and DDoS protection) to a **Hetzner VPS** running Ubuntu Server.
5.  **Reverse Proxy:** **Nginx** acts as a reverse proxy, receiving the HTTPS request, terminating the SSL, and securely forwarding the request to the Node.js application server running on a private port.
6.  **Application Logic:** The **PM2 Process Manager** runs and manages the **Node.js/Express.js** application. The Express server receives the filters, and a custom `buildMongoQuery` function translates the JSON into a complex, efficient MongoDB query.
7.  **Database Interaction:** The application connects to the **MongoDB** database instance (running securely on the same server) using credentials stored safely outside of version control. It executes the query against the indexed `supps` collection.
8.  **Response:** The API returns a JSON array of the matching supplements, projecting only the necessary fields for display. The React frontend then dynamically renders the results using a grid of `SupplementResultCard` components.

---

## **Technology Stack**

### **Frontend (Client-Side)**

*   **JavaScript (ES6+):** The primary language for all frontend logic.
*   **React (v19):** Leveraged for building a fast, component-based, and declarative UI. The application utilizes React 19's built-in support for rendering `<title>` and `<meta>` tags, simplifying SEO implementation.
*   **React Router:** Implemented for client-side routing between the Home and About pages.
*   **HTML5 & CSS3:** Used for structure and styling. The application features a fully responsive, mobile-first, modern dark theme built with pure CSS, utilizing Flexbox and Grid for complex layouts.
*   **No CSS Frameworks:** All styling is custom-written, demonstrating a strong command of modern CSS without reliance on libraries like Tailwind or Bootstrap.

### **Backend (Server-Side)**

*   **Node.js:** The JavaScript runtime environment used for the backend, enabling a full-stack JavaScript development experience.
*   **Express.js:** A minimal and flexible Node.js web application framework used to build the REST API.
*   **MongoDB:** The NoSQL database of choice, selected for its flexible, JSON-like document structure, which is ideal for easily adding new filterable fields and managing complex data arrays for ingredients and health goals.
*   **`mongodb` (Node.js Driver):** The official MongoDB driver used to connect to and interact with the database.
*   **`cors`:** Middleware used to enforce a strict Cross-Origin Resource Sharing policy, a critical security measure.
*   **`dotenv`:** Used to manage environment variables, keeping sensitive database credentials separate from the application code.

### **Infrastructure & Deployment (DevOps)**

*   **Hosting:**
    *   **Frontend:** Deployed on **Cloudflare Pages** for automated CI/CD, global distribution, and seamless integration with Cloudflare's security suite.
    *   **Backend & Database:** Hosted on a **Hetzner Virtual Private Server (VPS)** running **Ubuntu Server**.
*   **Web Server / Reverse Proxy:** **Nginx** is configured as a high-performance reverse proxy for SSL termination and routing requests to the Node.js application.
*   **Process Management:** **PM2** is used as a production-grade process manager for the Node.js API, ensuring the application remains active 24/7 and automatically restarts on failure or server reboot.
*   **Security:**
    *   **SSL/TLS:** End-to-end HTTPS encryption is enforced via **Let's Encrypt (Certbot)** on the server and Cloudflare's proxy.
    *   **CORS Policy:** The Express API is configured to accept requests exclusively from the `discoversupps.com` domain, preventing unauthorized API usage.
    *   **Database Isolation:** The MongoDB instance is bound to localhost and is not accessible from the public internet. The Node.js API is the sole entity authorized to communicate with it.
    *   **SSH Hardening:** The server is secured using key-based SSH authentication.

---

## **Core Technical Challenges & Solutions**

1.  #### **Ensuring Data Integrity and Eliminating Entry Errors**
    *   **Challenge:** The application's success hinges on perfect consistency between the frontend filter options and the data stored in the database. Manual data entry of hundreds of complex ingredient names is highly susceptible to spelling errors, which would break the filtering logic.
    *   **Solution:** I engineered a custom data entry tool using a separate, local React application. This "JSON Document Generator" presents all filterable fields (ingredients, health goals, etc.) as checkboxes with predefined, consistent spellings. This approach enforces data integrity at the source, completely eliminating the risk of typographical errors during data entry and ensuring that the data in the database always matches the frontend's filter values.

2.  #### **Translating Complex User Selections into an Efficient Database Query**
    *   **Challenge:** The filtering interface allows users to combine criteria from four distinct categories. The backend needed an intelligent way to translate this complex UI state into a single, performant MongoDB query that correctly handled different logical conditions (e.g., matching ANY selected form vs. matching ALL selected ingredients).
    *   **Solution:** I developed a dynamic query-building function (`buildMongoQuery`) in the Node.js backend. This function programmatically constructs a MongoDB query object. It uses the `$and` operator as the top-level structure and strategically applies different operators for each filter type: `$in` for supplement forms, dot notation for nested dietary restriction fields, and the powerful `$all` operator to ensure that a supplement contains every specified ingredient and health goal.

3.  #### **Designing a Secure, Production-Ready, Decoupled Architecture**
    *   **Challenge:** A key architectural requirement was to build a system where the sensitive database was completely isolated from the public internet while allowing a fast, globally-distributed frontend to access its data securely and efficiently.
    *   **Solution:** I designed and implemented a robust three-tier architecture. The React frontend is deployed on the edge via Cloudflare Pages. The Hetzner VPS hosts the Node.js API and the MongoDB database. Communication is strictly controlled: **Nginx** acts as a reverse proxy, handling all public traffic and forwarding it to the **PM2-managed** Node.js process, which is the only component with access to the database. A strict **CORS** policy on the API adds another layer of security, ensuring it only responds to requests from the legitimate frontend, creating a secure and scalable production environment.
