# Project Report: AI Resume Builder

## 1. Project Overview
*   **Goal:** AI Resume Builder is a sophisticated web application that leverages artificial intelligence to help users craft professional resumes. The application features an intuitive interface for creating, editing, and customizing resumes, along with robust backend services for secure data management and user authentication.
*   **Technologies Used:**
    - **Frontend:** React.js, TailwindCSS, Redux Toolkit, Vite, Framer Motion, React Router DOM
    - **Backend:** Node.js, Express.js, Docker, MongoDB (with Mongoose), JWT for authentication, bcrypt for password hashing
    - **AI Integration:** Google Generative AI (Gemini API) for smart resume suggestions
    - **UI Components:** Radix UI, Lucide React icons, React Draft WYSIWYG for rich text editing
    - **Other Tools:** Axios for API calls, UUID for unique identifiers, React Web Share for sharing functionality

## 2. Technical Implementation & Tutorial

### Architecture Overview
The application follows a client-server architecture with a React-based frontend and a Node.js/Express backend. The backend uses MongoDB as the database and implements RESTful APIs for data management. Authentication is handled via JWT tokens, and AI suggestions are powered by Google's Gemini API.

### Backend Structure
- **Entry Point:** `src/index.js` - Initializes the server, connects to MongoDB, and starts listening on the specified port.
- **App Configuration:** `src/app.js` - Sets up Express app with middleware (CORS, cookie parser, JSON parsing) and registers routes.
- **Database:** `src/db/index.js` - Handles MongoDB connection using Mongoose.
- **Models:** Located in `src/models/`, including:
  - `user.model.js` - User schema with authentication fields
  - `resume.model.js` - Main resume document schema
  - `education.model.js`, `experience.model.js`, `project.model.js`, `skill.model.js` - Component schemas for resume sections
- **Controllers:** `src/controller/` - Business logic for user and resume operations
  - `user.controller.js` - Handles user registration, login, and profile management
  - `resume.controller.js` - Manages CRUD operations for resumes and their components
- **Routes:** `src/routes/` - API endpoints
  - `user.routes.js` - Authentication and user-related routes
  - `resume.routes.js` - Resume management routes
- **Middleware:** `src/middleware/auth.js` - JWT authentication middleware
- **Utils:** `src/utils/` - Custom error and response classes for consistent API responses

### Frontend Structure
- **Entry Point:** `src/main.jsx` - Sets up React app with Redux store and routing
- **App Component:** `src/App.jsx` - Main app wrapper with routing configuration
- **Pages:** `src/pages/` - Route-based components
  - `home/HomePage.jsx` - Landing page
  - `auth/` - Authentication pages (Sign In, Sign Up)
  - `dashboard/Dashboard.jsx` - User dashboard for managing resumes
  - `dashboard/edit-resume/` - Resume editing interface with form and preview
  - `dashboard/view-resume/` - Resume viewing and export page
- **Components:** `src/components/` - Reusable UI components
  - `ui/` - Shadcn/ui components (buttons, dialogs, inputs, etc.)
  - `custom/` - Custom components like Header, RichTextEditor
- **Features:** `src/features/` - Redux slices for state management
  - `user/userFeatures.js` - User-related state
  - `resume/resumeFeatures.js` - Resume-related state
- **Services:** `src/Services/` - API integration
  - `GlobalApi.js` - Axios instance configuration
  - `resumeAPI.js` - Resume-related API calls
  - `AiModel.js` - AI suggestion service using Gemini API
  - `login.js` - Authentication API calls
- **Store:** `src/store/store.js` - Redux store configuration
- **Config:** `src/config/config.js` - Application configuration constants

### Key Code Snippets

#### Backend: User Authentication (src/controller/user.controller.js)
```javascript
import { User } from "../models/user.model.js";
import { ApiError } from "../utils/ApiError.js";
import { ApiResponse } from "../utils/ApiResponse.js";
import jwt from "jsonwebtoken";
import bcrypt from "bcrypt";

const generateAccessToken = (user) => {
  return jwt.sign(
    { _id: user._id, email: user.email },
    process.env.JWT_SECRET_KEY,
    { expiresIn: process.env.JWT_SECRET_EXPIRES_IN }
  );
};

export const registerUser = async (req, res) => {
  try {
    const { email, password } = req.body;
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      throw new ApiError(409, "User already exists");
    }
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = await User.create({ email, password: hashedPassword });
    const token = generateAccessToken(user);
    res.status(201).json(new ApiResponse(201, { user, token }, "User registered successfully"));
  } catch (error) {
    throw new ApiError(500, error.message);
  }
};
```

#### Frontend: Resume Form Component (src/pages/dashboard/edit-resume/components/form-components/PersonalDetails.jsx)
```jsx
import React, { useState } from 'react';
import { Input } from '../../../../components/ui/input';
import { Button } from '../../../../components/ui/button';

const PersonalDetails = ({ resumeInfo, setResumeInfo }) => {
  const [formData, setFormData] = useState(resumeInfo?.personalDetails || {});

  const handleChange = (field, value) => {
    setFormData({ ...formData, [field]: value });
    setResumeInfo({ ...resumeInfo, personalDetails: { ...formData, [field]: value } });
  };

  return (
    <div className="p-5 shadow-lg rounded-lg border-t-primary border-t-4 mt-10">
      <h2 className="font-bold text-lg">Personal Details</h2>
      <div className="grid grid-cols-2 mt-5 gap-3">
        <div>
          <label className="text-sm">First Name</label>
          <Input value={formData.firstName || ''} onChange={(e) => handleChange('firstName', e.target.value)} />
        </div>
        <div>
          <label className="text-sm">Last Name</label>
          <Input value={formData.lastName || ''} onChange={(e) => handleChange('lastName', e.target.value)} />
        </div>
        <div className="col-span-2">
          <label className="text-sm">Job Title</label>
          <Input value={formData.jobTitle || ''} onChange={(e) => handleChange('jobTitle', e.target.value)} />
        </div>
        <div className="col-span-2">
          <label className="text-sm">Address</label>
          <Input value={formData.address || ''} onChange={(e) => handleChange('address', e.target.value)} />
        </div>
        <div>
          <label className="text-sm">Phone</label>
          <Input value={formData.phone || ''} onChange={(e) => handleChange('phone', e.target.value)} />
        </div>
        <div>
          <label className="text-sm">Email</label>
          <Input value={formData.email || ''} onChange={(e) => handleChange('email', e.target.value)} />
        </div>
      </div>
      <div className="mt-3 flex justify-end">
        <Button>Save</Button>
      </div>
    </div>
  );
};

export default PersonalDetails;
```

### Setup Tutorial
1. Clone the repository: `git clone https://github.com/sahidrajaansari/ai-resume-builder.git`
2. Set up environment variables for backend (.env) and frontend (.env.local)
3. For Docker setup: Navigate to Backend/, run `docker-compose up -d`, then start frontend with `npm install && npm run dev`
4. For manual setup: Install dependencies in both Backend and Frontend directories, then run `npm run dev` in each

## 3. Project Management & Metrics
*   **Start Date:** [Not specified in available documentation - estimated based on repository creation]
*   **Contributors:** Mohamed Murshal Ibrahim (Lead Developer) - LinkedIn: https://www.linkedin.com/in/mohamedmurshalibrahim/
*   **Milestones Achieved:**
    - Implemented secure user authentication with JWT and bcrypt
    - Developed comprehensive resume builder with multiple sections (Personal Details, Education, Experience, Projects, Skills, Summary)
    - Integrated AI-powered suggestions using Google Gemini API
    - Created responsive UI with multiple theme options
    - Implemented live preview functionality
    - Added PDF export capability
    - Set up Docker containerization for backend
    - Deployed live demo on Netlify

## 4. Challenges and Learnings
*   **Challenge 1: AI Integration** - Integrating Google Gemini API for resume suggestions required careful handling of API keys and rate limiting. Resolution: Implemented proper error handling and fallback mechanisms for AI suggestions.
*   **Challenge 2: Real-time Preview** - Ensuring the resume preview updates in real-time while editing forms. Resolution: Used Redux for efficient state management and optimized re-rendering with React's useEffect and useState hooks.
*   **Challenge 3: PDF Export** - Generating high-quality PDFs from dynamic React components. Resolution: Utilized browser's print functionality with custom CSS for print media, ensuring proper formatting and layout.
*   **Challenge 4: Authentication Security** - Implementing secure JWT-based authentication across frontend and backend. Resolution: Used bcrypt for password hashing, implemented refresh token logic, and added middleware for route protection.
*   **Learning: Full-Stack Development** - Gained experience in coordinating frontend-backend interactions, API design, and database schema planning for a complete web application.

## 5. Future Enhancements
*   Additional resume templates and customization options
*   Advanced AI features like resume scoring and job matching
*   Collaboration features for team resume editing
*   Integration with LinkedIn and other professional platforms for data import
*   Mobile app development using React Native
*   Advanced analytics and user behavior tracking
*   Multi-language support for international users
*   Integration with job portals for direct application submission
