<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a id="readme-top"></a>
<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Don't forget to give the project a star!
*** Thanks again! Now go create something AMAZING! :D
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![Version][version-shield]][version-url]
[![Pull Requests][pr-shield]][pr-url]
[![Last Commit][commit-shield]][commit-url]
[![Release][release-shield]][release-url]
[![Code Size][code-size-shield]][code-size-url]
[![Top Language][top-language-shield]][top-language-url]
[![Languages][languages-shield]][languages-url]



<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/earlution/vwmp">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">Visual Workflow Management Platform (VWMP)</h3>

  <p align="center">
    A lightweight, plugin-based visual workflow management platform. Drag-and-drop workflow builder, framework-agnostic core, extensible via plugins.
    <br />
    <a href="https://github.com/earlution/vwmp"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/earlution/vwmp">View Demo</a>
    &middot;
    <a href="https://github.com/earlution/vwmp/issues/new?labels=bug&template=bug-report---.md">Report Bug</a>
    &middot;
    <a href="https://github.com/earlution/vwmp/issues/new?labels=enhancement&template=feature-request---.md">Request Feature</a>
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#documentation">Documentation</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#changelog">Changelog</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

VWMP is a lightweight, plugin-based visual workflow management platform that enables teams to build, execute, and monitor automation workflows through an intuitive drag-and-drop interface.

**Key Features:**
- 🎨 **Visual Workflow Designer** - Build workflows visually, no CLI required
- 🔌 **Plugin Architecture** - Extensible via custom step handlers and workflow types
- ⚡ **Lightweight Core** - Minimal overhead, framework-agnostic Python engine
- 🚀 **FastAPI Backend** - High-performance API layer (optional headless mode)
- 📊 **Real-time Monitoring** - Track workflow execution with live status updates
- 📝 **YAML/JSON Definitions** - Version-controlled, human-readable workflow configs

Perfect for release automation, documentation workflows, code quality checks, and any repeatable process that benefits from visual design and plugin extensibility.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



### Project Status

**Current Development State:**

VWMP is currently being developed as a sub-project within [Confidentia](https://github.com/earlution/confidentia), where it was originally created to support Confidentia's development workflows. The codebase currently resides in the Confidentia repository.

**Spin-Off in Progress:**

We are actively working to make VWMP a standalone, independent project. The goal is to extract VWMP from Confidentia and establish it as its own open-source project in this repository. This includes:

- Separating VWMP-specific code from Confidentia dependencies
- Establishing independent architecture and plugin system
- Creating standalone documentation and deployment guides
- Setting up independent CI/CD and release processes

**Why This Repository Exists:**

This repository serves as the destination for the VWMP spin-off. It currently contains:
- Complete architecture documentation and design decisions
- Project structure and setup files
- Comprehensive knowledge base (KB/) with all VWMP documentation
- Planning and preparation for the standalone release

Once the spin-off is complete, this repository will contain the full VWMP codebase and be ready for independent development and contributions.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



### Built With

* [![Python][Python.org]][Python-url]
* [![FastAPI][FastAPI.io]][FastAPI-url]
* [![React][React.js]][React-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

This is an example of how you may give instructions on setting up your project locally.
To get a local copy up and running follow these simple example steps.

### Prerequisites

* Python 3.8+
* pip (Python package manager)
* Node.js and npm (for frontend development)

### Installation

1. Clone the repo
   ```sh
   git clone https://github.com/earlution/vwmp.git
   cd vwmp
   ```
2. Install Python dependencies
   ```sh
   pip install -r requirements.txt
   ```
3. Install frontend dependencies (if applicable)
   ```sh
   npm install
   ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage

VWMP enables you to build and execute workflows through a visual designer interface. Here are some common use cases:

* **Release Automation** - Automate your software release process with visual workflow design
* **Documentation Workflows** - Generate and maintain documentation automatically
* **Code Quality Checks** - Run linting, testing, and quality gates as part of your workflow
* **CI/CD Pipelines** - Build continuous integration and deployment workflows visually
* **Custom Automation** - Create any repeatable process workflow with plugin extensibility

_For more examples and detailed documentation, please refer to the [Knowledge Base (KB/)](KB/) directory or the [Documentation Hub](KB/Documentation/Developer_Docs/vwmp/_index.md)_

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- DOCUMENTATION -->
## Documentation

Comprehensive documentation is available in the [`KB/`](KB/) directory:

### 📚 Quick Links
- **[Documentation Hub](KB/Documentation/Developer_Docs/vwmp/_index.md)** - Central navigation for all VWMP documentation
- **[User Guide](KB/Documentation/Developer_Docs/vwmp/vwmp-user-guide.md)** - Complete user guide covering dashboard, designer, and execution
- **[Tutorials](KB/Documentation/Developer_Docs/vwmp/)** - Step-by-step guides for building workflows
- **[Best Practices](KB/Documentation/Developer_Docs/vwmp/best-practices.md)** - Workflow design patterns and recommendations

### 🏗️ Architecture Documentation
- **[Platform Architecture](KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md)** - High-level architecture design
- **[Plugin Architecture](KB/Architecture/Standards_and_ADRs/E01-vwmp-plugin-architecture.md)** - Plugin system design
- **[C4 Architecture Views](KB/Architecture/)** - Context, Container, Component, and Runtime views

### 📋 Project Management
- **[Epic Documentation](KB/PM_and_Portfolio/epics/overview/Epic%2001/)** - Epic 01 development documentation
- **[Story Documentation](KB/PM_and_Portfolio/stories/overview/Epic%2001/)** - Individual story details

See the [KB/README.md](KB/README.md) for a complete overview of all available documentation.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- ROADMAP -->
## Roadmap

- [ ] Feature 1
- [ ] Feature 2
- [ ] Feature 3
    - [ ] Nested Feature

See the [open issues](https://github.com/earlution/vwmp/issues) for a full list of proposed features (and known issues).

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CHANGELOG -->
## Changelog

All notable changes to this project are documented in [CHANGELOG.md](CHANGELOG.md).

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

We welcome contributions from the community! Here's how you can help:

### How to Contribute

1. **Fork the Repository**
   - Click the "Fork" button at the top of this page
   - This creates a copy of the repository in your GitHub account

2. **Clone Your Fork**
   ```bash
   git clone https://github.com/your-username/vwmp.git
   cd vwmp
   ```

3. **Create a Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```
   - Use descriptive branch names (e.g., `feature/add-custom-plugin`, `fix/workflow-validation`)

4. **Make Your Changes**
   - Follow the project's code style and conventions
   - Add tests for new features
   - Update documentation as needed
   - Ensure all tests pass

5. **Commit Your Changes**
   ```bash
   git commit -m 'Add: description of your changes'
   ```
   - Use clear, descriptive commit messages
   - Follow [Conventional Commits](https://www.conventionalcommits.org/) format when possible

6. **Push to Your Fork**
   ```bash
   git push origin feature/your-feature-name
   ```

7. **Open a Pull Request**
   - Go to the [original repository](https://github.com/earlution/vwmp)
   - Click "New Pull Request"
   - Select your fork and branch
   - Fill out the PR template with details about your changes
   - Link any related issues

### Contribution Guidelines

- **Issues**: Before starting work, check existing issues or open a new one to discuss your proposed changes
- **Code Quality**: Ensure your code follows Python best practices and passes linting
- **Documentation**: Update relevant documentation in the `KB/` directory
- **Tests**: Add tests for new features and ensure all existing tests pass
- **Changelog**: Update `CHANGELOG.md` with your changes under `[Unreleased]`

### Getting Help

- **Questions?** Open a [discussion](https://github.com/earlution/vwmp/discussions)
- **Found a Bug?** Open an [issue](https://github.com/earlution/vwmp/issues/new?labels=bug)
- **Have a Feature Idea?** Open an [issue](https://github.com/earlution/vwmp/issues/new?labels=enhancement)

Don't forget to give the project a star if you find it useful! ⭐

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Ruari Mears - earlution@gmail.com

Project Link: [https://github.com/earlution/vwmp](https://github.com/earlution/vwmp)

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

Use this space to list resources you find helpful and would like to give credit to. I've included a few of my favorites to kick things off!

* [Choose an Open Source License](https://choosealicense.com)
* [GitHub Emoji Cheat Sheet](https://www.webpagefx.com/tools/emoji-cheat-sheet)
* [Malven's Flexbox Cheatsheet](https://flexbox.malven.co/)
* [Malven's Grid Cheatsheet](https://grid.malven.co/)
* [Img Shields](https://shields.io)
* [GitHub Pages](https://pages.github.com)
* [Font Awesome](https://fontawesome.com)
* [React Icons](https://react-icons.github.io/react-icons/search)
* [Best README Template](https://github.com/othneildrew/Best-README-Template)
* [Keep a Changelog](https://keepachangelog.com/)

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/earlution/vwmp.svg?style=for-the-badge
[contributors-url]: https://github.com/earlution/vwmp/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/earlution/vwmp.svg?style=for-the-badge
[forks-url]: https://github.com/earlution/vwmp/network/members
[stars-shield]: https://img.shields.io/github/stars/earlution/vwmp.svg?style=for-the-badge
[stars-url]: https://github.com/earlution/vwmp/stargazers
[issues-shield]: https://img.shields.io/github/issues/earlution/vwmp.svg?style=for-the-badge
[issues-url]: https://github.com/earlution/vwmp/issues
[license-shield]: https://img.shields.io/github/license/earlution/vwmp.svg?style=for-the-badge
[license-url]: https://github.com/earlution/vwmp/blob/main/LICENSE
[version-shield]: https://img.shields.io/badge/version-0.0.0%2B7-blue.svg?style=for-the-badge
[version-url]: https://github.com/earlution/vwmp/releases
[pr-shield]: https://img.shields.io/github/issues-pr/earlution/vwmp.svg?style=for-the-badge
[pr-url]: https://github.com/earlution/vwmp/pulls
[commit-shield]: https://img.shields.io/github/last-commit/earlution/vwmp.svg?style=for-the-badge
[commit-url]: https://github.com/earlution/vwmp/commits/main
[release-shield]: https://img.shields.io/github/v/release/earlution/vwmp.svg?style=for-the-badge
[release-url]: https://github.com/earlution/vwmp/releases
[code-size-shield]: https://img.shields.io/github/languages/code-size/earlution/vwmp.svg?style=for-the-badge
[code-size-url]: https://github.com/earlution/vwmp
[top-language-shield]: https://img.shields.io/github/languages/top/earlution/vwmp.svg?style=for-the-badge
[top-language-url]: https://github.com/earlution/vwmp
[languages-shield]: https://img.shields.io/github/languages/count/earlution/vwmp.svg?style=for-the-badge
[languages-url]: https://github.com/earlution/vwmp
[product-screenshot]: images/screenshot.png
[Python.org]: https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white
[Python-url]: https://www.python.org/
[FastAPI.io]: https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi&logoColor=white
[FastAPI-url]: https://fastapi.tiangolo.com/
[React.js]: https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB
[React-url]: https://reactjs.org/

