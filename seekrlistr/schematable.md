# Database Schema and Tables for the Seekr and Listr Platform using Supabase

This document provides the database schema and table definitions for the **Seekr and Listr** platform, designed to be implemented using [Supabase](https://supabase.com/), which is built on top of PostgreSQL. The schema includes tables for users, roles, properties, messages, favorites, and other entities essential for the platform's functionality.

---

## Table of Contents

1. [Overview](#overview)
2. [Entity-Relationship Diagram](#entity-relationship-diagram)
3. [Schema Definitions](#schema-definitions)
   - [1. Roles Table](#1-roles-table)
   - [2. Users Table](#2-users-table)
   - [3. User Roles Table](#3-user_roles-table)
   - [4. Properties Table](#4-properties-table)
   - [5. Property Images Table](#5-property_images-table)
   - [6. Messages Table](#6-messages-table)
   - [7. Favorites Table](#7-favorites-table)
   - [8. Reviews Table](#8-reviews-table)
   - [9. Saved Searches Table](#9-saved_searches-table)
   - [10. Notifications Table](#10-notifications-table)
4. [SQL Scripts](#sql-scripts)
5. [Conclusion](#conclusion)

---

## Overview

The database schema is designed to support the following key features:

- **User Management**: Authentication, profiles, roles (Seekr, Listr, Admin).
- **Property Listings**: Creation and management of property listings by Listrs.
- **Search and Discovery**: Seekrs can search for properties with various filters.
- **Messaging**: In-app communication between Seekrs and Listrs.
- **Favorites and Saved Searches**: Seekrs can save properties and searches.
- **Reviews and Ratings**: Seekrs can leave reviews for properties and Listrs.
- **Notifications**: Real-time updates for messages, new listings, etc.

---

## Entity-Relationship Diagram

[![ER Diagram](https://example.com/er-diagram.png)](https://example.com/er-diagram.png)

*(Note: Replace the image link with the actual ER diagram URL if available.)*

---

## Schema Definitions

### 1. **Roles Table**

Stores the different roles that users can have.

- **Table Name**: `roles`

#### Columns:

- `id` (UUID): Primary Key
- `name` (Text): Role name (e.g., 'Seekr', 'Listr', 'Admin')
- `description` (Text): Description of the role

### 2. **Users Table**

Stores user account information.

- **Table Name**: `users`

#### Columns:

- `id` (UUID): Primary Key
- `email` (Text): Unique, not null
- `password_hash` (Text): Hashed password
- `first_name` (Text)
- `last_name` (Text)
- `profile_picture` (Text): URL to the profile picture
- `created_at` (Timestamp): Default now()
- `updated_at` (Timestamp): Default now()
- `is_active` (Boolean): Default true

### 3. **User_Roles Table**

Associates users with their roles, allowing for multiple roles per user.

- **Table Name**: `user_roles`

#### Columns:

- `user_id` (UUID): Foreign Key to `users(id)`
- `role_id` (UUID): Foreign Key to `roles(id)`
- `assigned_at` (Timestamp): Default now()

#### Constraints:

- Primary Key: (`user_id`, `role_id`)
- Foreign Keys: `user_id` references `users(id)`, `role_id` references `roles(id)`

### 4. **Properties Table**

Stores property listings created by Listrs.

- **Table Name**: `properties`

#### Columns:

- `id` (UUID): Primary Key
- `listr_id` (UUID): Foreign Key to `users(id)`
- `title` (Text)
- `description` (Text)
- `price` (Numeric)
- `address` (Text)
- `city` (Text)
- `state` (Text)
- `zip_code` (Text)
- `country` (Text)
- `latitude` (Numeric)
- `longitude` (Numeric)
- `property_type` (Text)
- `bedrooms` (Integer)
- `bathrooms` (Integer)
- `square_feet` (Integer)
- `amenities` (Text[]): Array of amenities
- `status` (Text): e.g., 'Available', 'Pending', 'Sold'
- `created_at` (Timestamp): Default now()
- `updated_at` (Timestamp): Default now()

#### Constraints:

- Foreign Key: `listr_id` references `users(id)`

### 5. **Property_Images Table**

Stores images associated with properties.

- **Table Name**: `property_images`

#### Columns:

- `id` (UUID): Primary Key
- `property_id` (UUID): Foreign Key to `properties(id)`
- `image_url` (Text)
- `uploaded_at` (Timestamp): Default now()

#### Constraints:

- Foreign Key: `property_id` references `properties(id)`

### 6. **Messages Table**

Stores messages exchanged between users.

- **Table Name**: `messages`

#### Columns:

- `id` (UUID): Primary Key
- `sender_id` (UUID): Foreign Key to `users(id)`
- `receiver_id` (UUID): Foreign Key to `users(id)`
- `property_id` (UUID): Foreign Key to `properties(id)` (optional)
- `content` (Text)
- `sent_at` (Timestamp): Default now()
- `is_read` (Boolean): Default false

#### Constraints:

- Foreign Keys:
  - `sender_id` references `users(id)`
  - `receiver_id` references `users(id)`
  - `property_id` references `properties(id)`

### 7. **Favorites Table**

Stores properties that Seekrs have favorited.

- **Table Name**: `favorites`

#### Columns:

- `user_id` (UUID): Foreign Key to `users(id)`
- `property_id` (UUID): Foreign Key to `properties(id)`
- `favorited_at` (Timestamp): Default now()

#### Constraints:

- Primary Key: (`user_id`, `property_id`)
- Foreign Keys:
  - `user_id` references `users(id)`
  - `property_id` references `properties(id)`

### 8. **Reviews Table**

Stores reviews and ratings given by Seekrs to properties or Listrs.

- **Table Name**: `reviews`

#### Columns:

- `id` (UUID): Primary Key
- `reviewer_id` (UUID): Foreign Key to `users(id)`
- `listr_id` (UUID): Foreign Key to `users(id)` (optional)
- `property_id` (UUID): Foreign Key to `properties(id)` (optional)
- `rating` (Integer): e.g., 1 to 5
- `comment` (Text)
- `created_at` (Timestamp): Default now()

#### Constraints:

- Foreign Keys:
  - `reviewer_id` references `users(id)`
  - `listr_id` references `users(id)`
  - `property_id` references `properties(id)`

### 9. **Saved_Searches Table**

Stores search criteria saved by Seekrs.

- **Table Name**: `saved_searches`

#### Columns:

- `id` (UUID): Primary Key
- `user_id` (UUID): Foreign Key to `users(id)`
- `search_name` (Text)
- `search_criteria` (JSONB): Stores search parameters
- `created_at` (Timestamp): Default now()

#### Constraints:

- Foreign Key: `user_id` references `users(id)`

### 10. **Notifications Table**

Stores notifications for users.

- **Table Name**: `notifications`

#### Columns:

- `id` (UUID): Primary Key
- `user_id` (UUID): Foreign Key to `users(id)`
- `type` (Text): e.g., 'Message', 'New Listing', 'Price Change'
- `data` (JSONB): Additional data related to the notification
- `is_read` (Boolean): Default false
- `created_at` (Timestamp): Default now()

#### Constraints:

- Foreign Key: `user_id` references `users(id)`

---

## SQL Scripts

Below are the SQL scripts to create the tables in Supabase (PostgreSQL). You can execute these scripts in the Supabase SQL editor.

### **1. Roles Table**

```sql
CREATE TABLE public.roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL UNIQUE,
  description TEXT
);