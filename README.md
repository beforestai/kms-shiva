# KMS Shivathmika

Knowledge Management System prototype and planning repository.

## Overview

This repository contains the current KMS prototype, logic mapping, and research material. It will be used as the base repository for future frontend, backend, and deployment work.

## Current Contents

- `prototype/` - HTML prototype and logic mapping files
- `research/` - Research notes, references, and discovery material
- `docs/` - Architecture notes, planning documents, and technical references

## Prototype Files

The current prototype includes:

- `kms-prototype.html` - HTML version of the KMS prototype
- `KMS-prototype-logic-mapping.md` - Logic mapping and flow documentation for the prototype

## Planned Architecture

The KMS is expected to have two major parts:

### Frontend

- Landing page
- Login page
- Chat UI
- Response display
- Citation links
- Create page

### Backend

- User accounts and login tracking
- PostgreSQL database
- Vector database using pgvector
- Open-source embedding model
- Low-cost LLM integration
- Chat history and usage tracking

## Deployment Plan

The prototype may be deployed later using Coolify.

For stable and predictable deployment, the project may include:

- `Dockerfile`
- `package.json`
- `server.js`
- `/healthz` health check route

## Status

This repository is currently in the prototype and planning stage. Production frontend, backend, database, and deployment work will be added in later phases.
