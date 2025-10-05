# Chat AI with Calendar - User Guide for Pulling and Running the Docker Image

This guide is for users who want to pull and run the Chat AI with Calendar application from DockerHub. Follow these instructions to get the application up and running on your system.

## Prerequisites

Before you begin, ensure you have the following installed:
- Docker Desktop / Docker Engine (version 24 or higher)
- Docker Compose V2

## Files You Need to Download

To run the application, you need to download these files:

1. [docker-compose.yml] - Main Docker Compose file that defines the services
2. [.env.example]- Template for environment variables (rename to `.env` and configure)

You can download these files directly from the repository or create them manually using the content provided below.

## Step-by-Step Instructions

### 1. Pull the Docker Image

First, pull the latest version of the application from DockerHub:

```bash
docker pull m0hamedabed/chat-ai-combined:latest
```

### 2. Create the Required Files

Create the following files in a new directory:

#### File 1: docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    image: m0hamedabed/chat-ai-combined:latest
    container_name: chat-ai-combined-app
    ports:
      - "80:80"
    environment:
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      MONGO_URI: ${MONGO_URI:-mongodb://admin:password@mongodb:27017/chatbot_db?authSource=admin}
      CALENDAR_URL: ${CALENDAR_URL}
      CALENDAR_USERNAME: ${CALENDAR_USERNAME}
      CALENDAR_PASSWORD: ${CALENDAR_PASSWORD}
    restart: unless-stopped
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - chat_ai
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  mongodb:
    image: mongo:5.0
    container_name: chatbot_mongodb
    restart: unless-stopped
    ports:
      - "${MONGODB_PORT:-27017}:27017"
    volumes:
      - mongodb_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGODB_ROOT_USERNAME:-admin}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGODB_ROOT_PASSWORD:-password}
      MONGO_INITDB_DATABASE: chatbot_db
    healthcheck:
      test: ["CMD", "mongosh", "--quiet", "--eval", "db.runCommand({ ping: 1 })"]
      interval: 30s
      timeout: 10s
      start_period: 30s
      retries: 5
    networks:
      - chat_ai

volumes:
  mongodb_data:

networks:
  chat_ai:
    driver: bridge
```

#### File 2: .env (based on .env.example)

Create a file named `.env` with the following content and update the values:

```env
# Chat AI with Calendar - Environment Variables

# Google Gemini API Key (required)
GEMINI_API_KEY=your_google_gemini_api_key_here

# MongoDB Connection String (required)
# You can use the default MongoDB container or provide your own connection string
MONGO_URI=mongodb://admin:password@mongodb:27017/chatbot_db?authSource=admin

# Calendar Integration (optional)
# Uncomment and configure if you want to use calendar features
# CALENDAR_URL=https://your-calendar-service.com
# CALENDAR_USERNAME=your_calendar_username
# CALENDAR_PASSWORD=your_calendar_password

# MongoDB Configuration (optional - change if needed)
MONGODB_PORT=27017
MONGODB_ROOT_USERNAME=admin
MONGODB_ROOT_PASSWORD=your_secure_password
```

### 3. Configure Environment Variables

Edit the `.env` file to add your specific configuration:

1. **GEMINI_API_KEY**: Obtain a Google Gemini API key from the Google Cloud Console
2. **MONGO_URI**: If you want to use an external MongoDB, update this with your connection string
3. **CALENDAR_URL, CALENDAR_USERNAME, CALENDAR_PASSWORD**: If you want to use calendar features, uncomment and configure these

### 4. Run the Application

Start the application using Docker Compose:

```bash
docker-compose up -d
```

This command will:
- Start the Chat AI with Calendar application container
- Start a MongoDB container for data storage
- Create a network for the containers to communicate
- Map port 80 on your host to port 80 in the container

### 5. Access the Application

Once the containers are running, you can access the application in your web browser:

- **Frontend**: `http://localhost`
- **API Endpoints**: Available at `http://localhost/api/`

### 6. Verify the Application is Running

Check that the containers are running:

```bash
docker-compose ps
```

You should see both the `chat-ai-combined-app` and `chatbot_mongodb` containers in the "Up" state.

Check the logs if needed:

```bash
docker-compose logs app
```

## Updating to the Latest Version

To update to the latest version of the application:

```bash
# Pull the latest image
docker pull m0hamedabed/chat-ai-combined:latest

# Stop and remove the current containers
docker-compose down

# Start the application with the new image
docker-compose up -d
```

## Stopping the Application

To stop the application:

```bash
docker-compose down
```

This will stop and remove the containers but keep the MongoDB data volume.

## Troubleshooting

### Common Issues

1. **Port Conflicts**: If port 80 is already in use, change the port mapping in `docker-compose.yml`:
   ```yaml
   ports:
     - "8080:80"  # Change host port to 8080
   ```
   Then access the application at `http://localhost:8080`

2. **Permission Issues**: Ensure Docker is running with appropriate permissions.

3. **MongoDB Connection Issues**: Check that the MongoDB container is healthy and the connection string is correct.

4. **API Key Issues**: Verify that your Google Gemini API key is valid and has the necessary permissions.

### Checking Container Status

```bash
# List running containers
docker ps

# Check logs for the application
docker logs chat-ai-combined-app

# Check logs for MongoDB
docker logs chatbot_mongodb
```

## Application Features

The Chat AI with Calendar application includes:

- **AI Chat**: Google Gemini powered conversations with chat history
- **User Management**: Registration, login, and profile management
- **Appointment Scheduling**: Book, reschedule, and cancel appointments
- **Calendar Integration**: Optional CalDAV integration for shared calendar
- **Responsive UI**: Modern React-based frontend

## API Endpoints

The application provides several REST API endpoints:

- `/api/register` - User registration
- `/api/login` - User authentication
- `/api/chat` - Chat with the AI
- `/api/appointments` - Manage appointments
- `/api/profile` - User profile management

Refer to the main documentation for a complete API reference.
