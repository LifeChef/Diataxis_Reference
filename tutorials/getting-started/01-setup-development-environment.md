# Tutorial: Setup Development Environment

> **Document Type**: Tutorial (Learning-Oriented)  
> **Audience**: Beginner  
> **Estimated Time**: 20 minutes  
> **Last Updated**: 2024

## Overview

In this tutorial, you'll set up a complete development environment for our project. By the end, you'll have all the tools installed and configured to start building features.

**What you'll learn:**
- How to install required development tools
- How to configure your local environment
- How to verify your setup is working correctly

**What you'll build:**
A fully functional local development environment ready for coding.

---

## Prerequisites

Before starting this tutorial, ensure you have:

- [ ] A computer with macOS, Windows, or Linux
- [ ] Administrator/sudo access on your machine
- [ ] Internet connection for downloading tools
- [ ] At least 5GB of free disk space

---

## Step 1: Install Git

Git is our version control system. Let's install it first.

### Instructions

1. **Check if Git is already installed**
   
   Open your terminal and run:
   
   ```bash
   git --version
   ```
   
   **Expected result**: If you see a version number (e.g., `git version 2.39.0`), Git is installed. Skip to Step 2.

2. **Install Git** (if not installed)
   
   **For macOS:**
   ```bash
   brew install git
   ```
   
   **For Linux (Ubuntu/Debian):**
   ```bash
   sudo apt-get update
   sudo apt-get install git
   ```
   
   **For Windows:**
   Download from https://git-scm.com/download/win and run the installer.

3. **Verify installation**
   
   ```bash
   git --version
   ```

### Checkpoint ✓

At this point, you should have:
- [ ] Git installed and showing a version number
- [ ] Terminal/command prompt working

---

## Step 2: Install Node.js and npm

Node.js is our JavaScript runtime, and npm is the package manager.

### Instructions

1. **Download and install Node.js**
   
   Visit https://nodejs.org and download the LTS version (v18 or higher).
   
   Run the installer and follow the prompts.

2. **Verify installation**
   
   ```bash
   node --version
   npm --version
   ```
   
   **Expected result**: 
   ```
   v18.17.0  (or higher)
   9.6.7     (or higher)
   ```

### Checkpoint ✓

You should now have:
- [ ] Node.js installed (v18+)
- [ ] npm installed (v9+)

---

## Step 3: Clone the Project Repository

Now let's get the actual project code.

### Instructions

1. **Create a workspace directory**
   
   ```bash
   mkdir ~/projects
   cd ~/projects
   ```

2. **Clone the repository**
   
   ```bash
   git clone https://github.com/your-org/your-project.git
   cd your-project
   ```
   
   **Expected result**: You should see files being downloaded and a new directory created.

3. **Verify the clone**
   
   ```bash
   ls -la
   ```
   
   You should see project files including `package.json`.

### Checkpoint ✓

- [ ] Repository cloned successfully
- [ ] You're in the project directory
- [ ] You can see project files

---

## Step 4: Install Project Dependencies

Install all the required packages for the project.

### Instructions

1. **Install dependencies**
   
   ```bash
   npm install
   ```
   
   This will take a few minutes. You'll see progress indicators.
   
   **Expected result**: Completion message with no errors.

2. **Verify installation**
   
   ```bash
   npm list --depth=0
   ```
   
   You should see a list of installed packages.

### Checkpoint ✓

- [ ] All dependencies installed
- [ ] No error messages
- [ ] `node_modules/` directory exists

---

## Step 5: Configure Environment Variables

Set up your local configuration.

### Instructions

1. **Copy the example environment file**
   
   ```bash
   cp .env.example .env
   ```

2. **Edit the .env file**
   
   Open `.env` in your text editor and set:
   
   ```
   NODE_ENV=development
   PORT=3000
   DATABASE_URL=postgresql://localhost:5432/devdb
   ```

3. **Verify the file**
   
   ```bash
   cat .env
   ```

### Checkpoint ✓

- [ ] `.env` file created
- [ ] Environment variables configured

---

## Step 6: Start the Development Server

Let's make sure everything works!

### Instructions

1. **Start the server**
   
   ```bash
   npm run dev
   ```
   
   **Expected output**:
   ```
   Server running on http://localhost:3000
   Development mode enabled
   ```

2. **Test in browser**
   
   Open http://localhost:3000 in your browser.
   
   You should see the application homepage.

3. **Celebrate!** 🎉
   
   Your development environment is ready!

---

## Summary

Congratulations! You've set up your development environment.

**You've learned:**
- How to install Git, Node.js, and npm
- How to clone a repository
- How to install project dependencies
- How to configure environment variables
- How to start the development server

**You've built:**
- A complete, working local development environment

---

## Next Steps

Now that your environment is ready:

1. **Continue learning**: Try [Your First Feature](02-your-first-feature.md)
2. **Explore the code**: Look through the project structure
3. **Read documentation**: Check the [Architecture Overview](../../reference/architecture/system-components.md)

---

## Troubleshooting

### Problem: "git: command not found"
**Solution**: Git isn't installed. Go back to Step 1 and install it.

### Problem: "npm install" fails
**Solution**: Check your Node.js version. You need v18 or higher.

### Problem: Port 3000 already in use
**Solution**: Change PORT in `.env` to 3001 or kill the process using port 3000.

---

**Author**: Documentation Team  
**Version**: 1.0
