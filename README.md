# 📄 Technical Requirement Document (TRD)

## 🎯 Project Name: Story-Based Microservices Platform  
**Date:** April 2025  
**Author:** Kanisius Wahyu Santoso

---

## 1. Overview

This platform consists of four main microservices: `Account`, `Notification`, `Story`, and `Comments`.  
Each service is responsible for its own domain and communicates with others via gRPC.  
User-facing endpoints are exposed through REST APIs.

---

## 2. Architecture Diagram

![image](https://github.com/user-attachments/assets/efdc1427-0bb8-4c36-859f-8d2b63292950)

---

## 3. Services

![Comment-Service](https://github.com/CelticAlreadyUse/Article-Comment-Service)
![Account-Service](https://github.com/CelticAlreadyUse/Article-accountservices)
![Story-Service](https://github.com/CelticAlreadyUse/article-story-service)
![Notification-Service](https://github.com/CelticAlreadyUse/Article-Notification-Services)

### 3.1 Account Service

- Manages user registration, authentication, email validation and profile data.
- **Database:** Mysql or MongoDB
- **External Interface:** REST API

#### gRPC Functions:

| Function Name        | Called By       | Description                                          |
|----------------------|-----------------|------------------------------------------------------|
| `NotifyUserCreation` | Notification    | Notifies Notification Service of a newly created user. |
| `GetUserInfo`        | Story, Comments | Fetches user data by user ID.                       |

---

### 3.2 Notification Service

- Handles all in-app, email, or push notifications.
- **Database:** 
- **External Interface:** None (internal gRPC only)

#### gRPC Functions:

| Function Name                     | Called By     | Description                                         |
|----------------------------------|---------------|-----------------------------------------------------|
| `SendNotification`               | Story, Comments | Sends notification to a specific user.              |
| `NotifyUserCreation`            | Account        | Triggered when a new user is registered.            |
| `NotifyNewComment`              | Comments       | Triggered when a story receives a new comment.      |
| `NotifyAuthorOnComment`         | Comments       | Notifies the author when someone comments on their story. |

---

### 3.3 Story Service

- Manages story creation, retrieval, and caching.
- **Database:** PostgreSQL or MongoDB  
- **Cache:** Redis  
- **External Interface:** REST API

#### gRPC Functions:

| Function Name            | Called By     | Description                                       |
|--------------------------|---------------|---------------------------------------------------|
| `GetStoriesByUser`       | Frontend      | Retrieves all stories by a specific user.         |
| `GetStoryByID`           | Comments      | Retrieves detailed story information.             |
| `NotifyCommented`        | Notification  | Notifies that the story has received a comment.   |
| `CacheStory`             | Internal      | Saves story content to Redis for fast access.     |

---

### 3.4 Comments Service

- Manages user comments on stories.
- **Database:** PostgreSQL or MongoDB  
- **External Interface:** REST API

#### gRPC Functions:

| Function Name                | Called By     | Description                                     |
|------------------------------|---------------|-------------------------------------------------|
| `AddComment`                 | Frontend      | Adds a new comment to a story.                  |
| `GetCommentsByStoryID`       | Frontend      | Fetches comments associated with a story.       |
| `NotifyNewComment`           | Notification  | Sends notification when a new comment is added. |

---

## 4. Data Flow Example: Story Creation

### 📘 User creates a new story

1. `POST /v1/stories` is sent to Story Service.
2. Story Service saves the story to the database.
3. Story Service calls `NotifyStoryPosted()` in Notification Service.
4. Notification Service sends notifications to the author's followers.

---

## 5. Service Dependencies

| Service      | Depends On              |
|--------------|--------------------------|
| Account      | Notification             |
| Notification | Account, Story, Comments |
| Story        | Notification, Account    |
| Comments     | Notification, Story, Account |

---

## 6. Non-Functional Requirements

- All inter-service communication must use gRPC.
- Redis will be used as a caching layer in the Story Service.
- External user requests should only be handled through REST APIs.
- All services must log critical events, including errors and triggered notifications.

---
