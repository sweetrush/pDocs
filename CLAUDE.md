# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pDocs is a comprehensive technical documentation repository organized as a personal knowledge base focused on software development, DevOps, and emerging technologies. The project serves as a digital island for sharing technical write-ups, guides, and learning resources across multiple domains.

## Architecture and Code Structure

### Project Structure
```
pDocs/
├── README.md              # Main project overview and navigation
├── SUMMARY.md             # Table of contents and directory structure
├── Blockchain/            # Blockchain development resources
├── DevOps/               # DevOps practices and tools
├── DockerWU/             # Docker write-ups and tutorials
├── LLMandAI/             # LLM and AI documentation
├── LinuxWriteUps/        # Linux system administration guides
├── Python/               # Python programming resources
├── SecArea/              # Security and cybersecurity topics
├── SteamlitArea/         # Streamlit framework documentation
├── TransformingDigitally/ # Digital transformation resources
├── Webdev/               # Web development resources
├── Images/               # Supporting images and diagrams
├── dev-ops-area/         # DevOps setup and environment
├── docker-write-ups/     # Additional Docker documentation
├── future-projects/      # Project roadmaps and plans
├── linux-write-ups/      # Additional Linux guides
├── llms-and-ai/          # Additional AI/LLM content
├── python/               # Additional Python resources
├── security-area/        # Additional security content
├── steamlit-area/        # Additional Streamlit content
├── transforming-digitally/ # Additional digital transformation content
└── blockchain/           # Additional blockchain content
```

### Content Architecture
The repository follows a hierarchical documentation structure with:
- **Root README**: Project overview and main content areas
- **Category READMEs**: Brief introductions to each technical area
- **Individual write-ups**: Comprehensive guides in Markdown format
- **SUMMARY.md**: Central navigation linking all documentation

### Content Categories
1. **DevOps**: Git workflows, virtual environments, development practices
2. **Python**: Programming tutorials, virtual environment setup
3. **Docker**: Containerization, networking, service deployment
4. **Linux System Administration**: SSH, CLI tools, system configuration
5. **LLMs and AI**: Prompt engineering, local LLM deployment, RAG
6. **Blockchain**: Blockchain fundamentals, NFT development
7. **Cybersecurity**: Security tools, architecture, penetration testing
8. **Streamlit**: Python web framework for data applications
9. **Web Development**: Web technologies and development practices
10. **Digital Transformation**: Cloud solutions, data visualization

## Common Development Commands

### Git Operations
```bash
# Clone the repository
git clone git@github.com:sweetrush/pDocs.git

# Check repository status
git status

# Add files to staging
git add [filename]

# Commit changes
git commit -m "Descriptive commit message"

# Push to remote
git push

# Pull latest changes
git pull

# View commit history
git log --oneline --graph --decorate

# Create and switch to new branch
git switch -c feature-branch

# Merge branch
git merge feature-branch

# Rebase branch
git rebase

# Reset to previous commit
git reset --hard [commit-id]
```

### Navigation and Content Management
```bash
# List all directories
find /path/to/pDocs -type d | sort

# Count documentation files (currently 48 markdown files)
find /path/to/pDocs -name "*.md" | wc -l

# Find longest documentation files
find /path/to/pDocs -name "*.md" -exec wc -l {} \; | sort -n | tail -10

# Check recent commits
git log --oneline -10

# View remote configuration
git remote -v
```

## Key Components and Architecture Patterns

### 1. Documentation Standards
- **Markdown Format**: All documentation uses standard Markdown syntax
- **Hierarchical Organization**: Categories contain subdirectories with related content
- **Cross-Referencing**: SUMMARY.md provides centralized navigation
- **Image Support**: Images directory for supporting visual content

### 2. Content Categories Structure
Each technical area follows a consistent pattern:
- **README.md**: Brief introduction to the category
- **Subdirectories**: Organized write-ups and tutorials
- **Practical Guides**: Step-by-step instructions with code examples
- **Best Practices**: Industry standards and recommendations

### 3. Git Workflow Pattern
- **Standard Git Operations**: Commit, push, pull, branch management
- **Branch Strategy**: Feature branches for new content development
- **Commit History**: Descriptive messages with clear intent
- **Remote Management**: GitHub integration for collaboration

### 4. Content Creation Patterns
- **Step-by-Step Tutorials**: Commands and code examples
- **Screenshot Integration**: Images for visual guidance
- **Code Snippets**: Syntax-highlighted code examples
- **Cross-References**: Links to related documentation

## Important Configuration Files and Patterns

### 1. Central Navigation Files
- **`README.md`**: Main project overview and content areas
- **`SUMMARY.md`**: Comprehensive table of contents and navigation structure

### 2. Git Configuration
- **Remote Repository**: `git@github.com:sweetrush/pDocs.git`
- **Branch Strategy**: Standard Git branching with main/master branches
- **Commit History**: Detailed commit messages with clear intent

### 3. Content Organization Patterns
- **Directory Naming**: Lowercase with hyphens or underscores
- **File Naming**: Descriptive names with underscores
- **Content Structure**: Hierarchical by technical domain
- **Version Control**: Git-based content management

### 4. Image Management
- **Storage Location**: `/Images/0001/` directory
- **File Format**: PNG format for compatibility
- **Referencing**: Markdown image syntax in documentation

## Development Guidelines

### Content Creation Standards
1. **Markdown Format**: Use standard Markdown for all documentation
2. **Consistent Structure**: Follow established directory naming conventions
3. **Code Examples**: Include practical, working code examples
4. **Visual Support**: Add screenshots where helpful for clarity
5. **Cross-Referencing**: Link to related documentation in SUMMARY.md

### Git Best Practices
1. **Descriptive Commits**: Use clear, specific commit messages
2. **Branch Usage**: Create feature branches for new content
3. **Regular Commits**: Commit frequently with logical units of work
4. **Pull Requests**: Use GitHub's PR system for content review
5. **Main Branch**: Keep main branch stable and ready for deployment

### Quality Assurance
1. **Content Review**: Ensure technical accuracy and completeness
2. **Formatting Consistency**: Maintain consistent formatting throughout
3. **Image Quality**: Use clear, appropriately sized images
4. **Update Regularly**: Keep content current with latest technologies
5. **Error Checking**: Verify all code examples work as expected

## Integration Patterns

### 1. Cross-Category Integration
- **DevOps + Docker**: Container-based development workflows
- **Python + Streamlit**: Data applications and web interfaces
- **Security + All Categories**: Security best practices across technologies
- **Linux + All Categories**: System administration fundamentals

### 2. Learning Path Integration
- **Beginner**: Python, Linux basics, Git fundamentals
- **Intermediate**: Docker, DevOps, Web development
- **Advanced**: LLMs, Blockchain, Security architecture
- **Specialized**: Streamlit, Digital transformation

This documentation repository serves as a comprehensive learning resource and reference guide, with a focus on practical, hands-on technical knowledge across multiple domains of software development and IT operations.