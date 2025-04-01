# Zone Mapper Project Plan

## Project Overview

Zone Mapper is a web application that allows users to create, edit, label, and manage geographic zones on Google Maps. The application will provide an intuitive interface for defining custom zones, organizing them in a catalog, and storing them in a PostgreSQL database for persistence and retrieval.

## Technology Stack

- **Frontend**: Flutter Web
- **Backend**: Node.js with Express
- **Database**: PostgreSQL
- **Maps Integration**: Google Maps API
- **Authentication**: Firebase Authentication
- **State Management**: Provider/Riverpod
- **API Communication**: REST APIs

## Features

### Core Features

1. **Google Maps Integration**
   - Interactive map display
   - Custom zone drawing tools
   - Zoom and pan controls
   - Satellite/terrain view toggle

2. **Zone Management**
   - Create polygonal zones by placing points on the map
   - Edit existing zones (resize, reshape, reposition)
   - Delete zones (mark as deleted in the workflow)
   - Assign color and style to zones
   - Manage zone workflow status (in-progress, published, deleted)
   - Track status change history with audit trail

3. **Zone Metadata**
   - Label zones with custom names
   - Add descriptions to zones
   - Categorize zones with tags
   - Record creation and modification timestamps

4. **Zone Catalog**
   - List view of all created zones
   - Filtering and searching capabilities including by status
   - Sorting options (alphabetical, date created, size, status)
   - Status-based views (in-progress zones, published zones, etc.)
   - Pagination for large catalogs

5. **Data Import/Export**
   - Import zones from GeoJSON/KML
   - Export zones to GeoJSON/KML
   - Bulk zone operations

6. **User Management**
   - User registration and login
   - Role-based access control
   - Zone sharing between users

## System Architecture

### Frontend Architecture

```
zones_mapper_frontend/
├── lib/
│   ├── main.dart                  # Application entry point
│   ├── config/                    # App configuration (API keys, endpoints)
│   ├── models/                    # Data models for zones and metadata
│   ├── screens/                   # Application screens/pages
│   │   ├── auth/                  # Login/Register screens
│   │   ├── map/                   # Map editing screen
│   │   ├── catalog/               # Zone catalog management 
│   │   └── settings/              # User settings
│   ├── widgets/                   # Reusable UI components
│   │   ├── map_widgets/           # Map-specific widgets
│   │   ├── zone_widgets/          # Zone-specific widgets
│   │   └── common/                # Common UI elements
│   ├── services/                  # API services
│   │   ├── auth_service.dart      # Authentication service
│   │   ├── zone_service.dart      # Zone CRUD operations
│   │   └── map_service.dart       # Google Maps service
│   ├── utils/                     # Utility functions
│   └── state/                     # State management
```

### Backend Architecture

```
zones_mapper_backend/
├── src/
│   ├── server.js                  # Server entry point
│   ├── config/                    # Server configuration
│   ├── controllers/               # Request handlers
│   │   ├── zone_controller.js     # Zone CRUD operations
│   │   ├── auth_controller.js     # User authentication
│   │   └── import_export.js       # Import/export functionality
│   ├── models/                    # Database models
│   │   ├── zone.js                # Zone model
│   │   └── user.js                # User model
│   ├── routes/                    # API routes
│   ├── middleware/                # Custom middleware
│   └── utils/                     # Utility functions
├── migrations/                    # Database migrations
└── tests/                         # Backend tests
```

### Database Schema

**Users Table**
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  firebase_uid VARCHAR(128) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255),
  profile_photo_url TEXT,
  refresh_token TEXT,
  refresh_token_expires_at TIMESTAMP,
  last_login TIMESTAMP,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Roles Table**
```sql
CREATE TABLE roles (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert default roles
INSERT INTO roles (name, description) VALUES 
  ('admin', 'Administrator with full access to all features'),
  ('user', 'Standard user with access to own zones'),
  ('viewer', 'View-only access to shared zones');
```

**User_Roles Table**
```sql
CREATE TABLE user_roles (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, role_id)
);
```

**Zones Table**
```sql
CREATE TABLE zones (
  id SERIAL PRIMARY KEY,
  owner_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  boundary GEOMETRY(POLYGON) NOT NULL,
  color VARCHAR(7),
  is_public BOOLEAN DEFAULT FALSE,
  status VARCHAR(20) CHECK (status IN ('in-progress', 'published', 'deleted')) DEFAULT 'in-progress',
  status_changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status_changed_by INTEGER REFERENCES users(id),
  tags TEXT[],
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Zone_Status_History Table**
```sql
CREATE TABLE zone_status_history (
  id SERIAL PRIMARY KEY,
  zone_id INTEGER REFERENCES zones(id) ON DELETE CASCADE,
  previous_status VARCHAR(20) CHECK (previous_status IN ('in-progress', 'published', 'deleted')),
  new_status VARCHAR(20) CHECK (new_status IN ('in-progress', 'published', 'deleted')),
  changed_by INTEGER REFERENCES users(id) ON DELETE SET NULL,
  notes TEXT,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Zone_Permissions Table**
```sql
CREATE TABLE zone_permissions (
  id SERIAL PRIMARY KEY,
  zone_id INTEGER REFERENCES zones(id) ON DELETE CASCADE,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  can_view BOOLEAN DEFAULT FALSE,
  can_edit BOOLEAN DEFAULT FALSE,
  can_delete BOOLEAN DEFAULT FALSE,
  can_share BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(zone_id, user_id)
);
```

## Implementation Plan

### Phase 1: Setup and Infrastructure (Week 1-2)

1. **Project Setup**
   - Initialize Flutter web project
   - Set up Node.js backend with Express
   - Configure PostgreSQL database with PostGIS extension
   - Set up development environments
   - Configure CI/CD pipeline

2. **Authentication Implementation**
   - Implement user registration and login UI
   - Set up Firebase Authentication
   - Create user database tables
   - Implement JWT token authentication

### Phase 2: Core Map Functionality (Week 3-4)

1. **Google Maps Integration**
   - Integrate Google Maps API with Flutter
   - Implement basic map controls
   - Add zone drawing tools
   - Implement zone editing capabilities

2. **Backend Zone Management**
   - Create API endpoints for zone CRUD operations
   - Implement database operations for zones
   - Set up validation and error handling

### Phase 3: Zone Catalog and Metadata (Week 5-6)

1. **Zone Catalog UI**
   - Implement zone listing interface
   - Add filtering and sorting capabilities
   - Create zone detail view
   - Implement zone metadata editing
   - Add workflow status management UI
   - Implement status change history view

2. **Data Processing**
   - Implement GeoJSON/KML import functionality
   - Create export capabilities
   - Implement bulk operations

### Phase 4: Advanced Features and Polishing (Week 7-8)

1. **Advanced Features**
   - Implement zone sharing
   - Add role-based permissions

2. **Testing and Optimization**
   - Perform unit and integration testing
   - Optimize database queries
   - Improve UI/UX
   - Performance testing and optimization

### Phase 5: Deployment and Launch (Week 9)

1. **Deployment Preparation**
   - Finalize documentation
   - Prepare deployment scripts
   - Set up monitoring and logging

2. **Launch**
   - Deploy to production environment
   - Perform final testing
   - Monitor for issues
   - Collect user feedback

## Required Resources

### External APIs and Services

1. **Google Maps API**
   - Maps JavaScript API
   - Geocoding API
   - Places API

2. **Firebase**
   - Authentication
   - Cloud Functions (optional)

### Development Tools

1. **Development Environment**
   - Visual Studio Code / Windsurf / Cursor
   - Github for version control
   - Docker for containerization

2. **Testing Tools**
   - Flutter testing framework
   - Jest for backend testing
   - Postman for API testing

## Challenges and Considerations

1. **Technical Challenges**
   - Ensuring smooth performance of map drawing operations in Flutter Web
   - Handling large polygon datasets efficiently
   - Implementing real-time collaboration features

2. **Security Considerations**
   - Protecting API keys
   - Implementing proper authentication and authorization
   - Data privacy and access control

3. **Scalability**
   - Designing for potential growth in user base
   - Optimizing database for spatial queries
   - Caching strategies for frequently accessed data

## Next Steps

1. Set up the development environment
2. Create initial project structure
3. Implement basic authentication
4. Begin Google Maps integration

---

This project plan serves as a roadmap for developing the Zone Mapper application. The timeline and feature set may be adjusted based on development progress and additional requirements that emerge during implementation.
