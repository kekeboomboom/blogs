# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal knowledge base and blog repository containing technical documentation, essays, and learning notes organized in a hierarchical structure. The content is primarily in Markdown format and covers software engineering, algorithms, system design, and technical concepts.

## Architecture and Structure

### Content Organization
- `AIGC/` - AI and machine learning related content
- `Tech/` - Technical documentation organized by domain:
  - `Java/` - Java ecosystem (Spring, JVM tuning)
  - `cloud/` - Container technologies (Docker, Harbor, MinIO)
  - `database/` - Database technologies (MySQL, Redis)
  - `算法/` - Algorithms and data structures
  - `面经/` - Interview preparation materials
  - `分布式与微服务/` - Distributed systems and microservices
- `Essay/` - Personal essays and reflections
- `archive/` - Historical project documentation (Cogent platform)
- `sell/` - Book summaries and reviews
- `img/` - Image assets

### Content Types
All content is documentation-based using Markdown files. There are no build processes, dependency management, or automated tooling configured for this repository.

## Development Workflow

### Git Conventions
This repository follows [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#summary) for commit messages as referenced in the README.

### File Management
- All content files use `.md` extension
- Images are stored in the `img/` directory
- No package management or build tools are configured
- Content is version controlled using Git only

## Content Guidelines

### When Adding New Content
- Place technical documentation in appropriate `Tech/` subdirectories
- Use descriptive filenames that reflect the content topic
- Maintain the existing directory structure and organization
- Follow existing Markdown formatting conventions observed in other files

### File Organization Patterns
- Group related topics in subdirectories (e.g., `Java/spring/`, `算法/二叉树/`)
- Use clear, descriptive Chinese or English names for directories and files
- Archive outdated or project-specific content in `archive/`