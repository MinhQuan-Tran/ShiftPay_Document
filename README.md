# ShiftPay Documentation

Welcome to the documentation repository for ShiftPay. This repository contains all the detailed documentation related to the project, including system design, functional requirements, technical specifications, user stories, wireframes, and more.

## Project Overview

**ShiftPay** helps hourly and shift-based workers keep their schedules and earnings organised without spreadsheets or guesswork. It makes logging, planning, and reviewing work simple, whether online or offline.

### What It Solves

- **Fragmented tracking:** replaces scattered notes/apps with one simple place.
- **Uncertainty about hours:** clearly shows shift durations and totals.
- **Weekly planning:** visual schedule to plan and adjust quickly.
- **Connectivity gaps:** works offline; syncs when you sign in.
- **Data inconsistency:** consistent formats so your records stay clean.
### Target Audience

- **Shift Workers:** Individuals working hourly or irregular shifts who need to track both their time and earnings.
- **Freelancers:** Workers with flexible schedules and multiple employers who require accurate tracking of hours and pay.

### Key Features

- **Shift Tracking:** Log and manage shifts with start/end times, pay rate, breaks, and more.
- **Earnings Calculation:** Automatically compute pay based on hours worked and rates.
- **Offline-first:** Access and log shifts without internet; sync later.

[Live Version](https://shiftpay-mqtran.netlify.app/)

[Frontend Repository](https://github.com/MinhQuan-Tran/ShiftPay_Frontend)  
[Backend Repository](https://github.com/MinhQuan-Tran/ShiftPay_Backend)

## Table of Contents

- [Introduction](#introduction)
- [Functional Requirements](#functional-requirements)
- [System Architecture](#system-architecture)
- [API Design](#api-design)
- [Wireframes](#wireframes)
- [Testing](#testing)
- [License](#license)

## Introduction

This repository serves as a central place for all the documentation related to the ShiftPay. It provides an in-depth view of the functionality, architecture, and design decisions that went into building the project. The docs are structured as follows:

- **Functional Requirements**: Describes the key features and functionalities of the project.
- **Technical Design**: Provides details on how the system is implemented, including technology stack, database structure, and API design.
- **Wireframes**: Visual representations of the user interface (UI) design.
- **User Stories**: A breakdown of user interactions and use cases for the system.
- **API Design**: Details the API endpoints, request/response structures, and authentication methods used in the backend.

## Functional Requirements

The **Functional Requirements** section outlines the main features of the application. This includes:
- **Authentication:** Sign in/out via Azure AD B2C; UI reflects auth state.
- **Shifts Management:** Create, view, edit, delete shifts; daily/weekly views; validated data; duration calculation.
- **Shift Templates:** Create, edit, delete templates; apply templates to prefill new shifts.
- **Shift Recurrence:** Define recurring schedules; expand into dated instances; edit or skip single occurrences.
- **Persistence and Sync:** Store locally when signed out; sync with backend API when signed in.

Full [Functional Requirements document](docs/functional-requirements.md).

## System Architecture

This section use C4 Model provides a visual diagram and process flow of the system architecture, including the backend, frontend, and database interactions.

You can view the **System Architecture** diagram and details [here](https://s.icepanel.io/6Gly5HmgeLhqbo/1TFX).

## API Design

This section describes the API endpoints used in the project. It includes:
- RESTful API structure
- Detailed explanation of each API endpoint, its request and response formats, and any authentication methods (such as JWT).

For detailed API documentation, check out the **API Design** document [here](docs/api-design.md).

## Wireframes

Wireframes are provided to showcase the user interface design and flow. These wireframes help visualise the key screens of the application and user interactions.

You can view the **Wireframes** folder [here](docs/wireframes/).

## Testing

The **Testing** section outlines the various tests conducted for the project, including unit tests, integration tests, and end-to-end tests. It also includes instructions for setting up and running tests.

You can access the **Testing** documentation [here](docs/testing.md).

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.

---

### Contribution

If you would like to contribute to improving the documentation, feel free to create a pull request or open an issue.