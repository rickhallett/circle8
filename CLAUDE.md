# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Circle8 is a collection of AI instructor prompts and patterns for teaching various technical domains. The project focuses on creating specialized AI instructors that teach through pattern recognition, hands-on implementation, and business value connection.

## Repository Structure

- **Root Directory**: Contains specialized instructor prompt templates
  - `backend_instructor.md`: Backend SQL patterns master instructor
  - `core_instructor.md`: Core agentic systems architecture instructor framework
  - `orchestration_instructor.md`: Agent orchestration master instructor
  - `dev_complete_program.md`: Complete developer education program outline
- **ai_docs/**: Documentation for AI tools and services
  - Various tool documentation including Claude Code, web search, and agent frameworks

## Key Architectural Patterns

### Instructor Design Philosophy
- **Pattern-First Teaching**: Focus on reusable patterns over specific syntax
- **Business Value Connection**: Every technical pattern connects to measurable business outcomes
- **Hands-On Learning**: Concepts taught through practical coding exercises
- **Progressive Complexity**: Start simple, layer complexity gradually
- **Failure-Driven Learning**: Students learn by breaking systems and understanding failure modes

### Core Teaching Structure
Each instructor follows a consistent daily lesson structure:
1. Pattern Introduction (5-8 minutes)
2. Minimal Implementation (20-25 minutes)
3. Interactive Exploration/Stress Testing (12-15 minutes)
4. Real-World Connection (10 minutes)
5. Integration & Trade-offs (5-10 minutes)

## Development Guidelines

### When Creating New Instructors
- Include pre-lesson market checks using web search for current trends
- Structure content in progressive weekly modules
- Connect every pattern to business value and industry relevance
- Include explicit trade-off analysis for architectural decisions
- Focus on decision frameworks rather than absolute best practices

### Content Principles
- Teach patterns appropriate for actual business problems (not just Google-scale)
- Include cost consciousness in all architectural discussions
- Prepare students for emerging technologies and future patterns
- Use current tech stacks in examples (PostgreSQL, Redis, Node.js/Python/Go)
- Reference real companies and industries where patterns create value

### Assessment Through Real Problems
Students demonstrate mastery by:
- Choosing appropriate patterns for specific requirements
- Articulating trade-offs in architectural decisions
- Estimating costs and performance implications
- Adapting patterns when requirements change
- Designing for failure with recovery strategies

## Common Tasks

Since this is a documentation/instructional content repository, common tasks include:
- Adding new instructor templates
- Updating existing instructors with current market trends
- Refining teaching progressions based on effectiveness
- Adding supporting documentation to ai_docs/
- Creating specialized instructors for emerging technologies

## Important Notes

- This repository focuses on educational content and prompt engineering, not traditional code
- All instructors should maintain awareness of current technology trends through web searches
- Content should balance theoretical understanding with practical implementation
- Focus on patterns that create measurable business value in production systems