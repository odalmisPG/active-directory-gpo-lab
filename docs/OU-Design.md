# OU Design: myITlab.local

## Goal

Model a small company with separate OUs for computers, servers, and user departments to make Group Policy targeting clear and realistic.

## OU Structure

- LAB-AD (root OU)
  - Computers
    - Servers
       - DomainControllers
       - Member Servers
    - Workstations
      - Windows
      - Linux
  - Groups
  - Service Accounts
  - Users
    - Customer Service
    - Human Resources
    - IT
    - Operations
    - Sales
    - Admin Accounts

## Why This Design

- Separate OUs for servers vs workstations to apply different hardening GPOs.
- Department-based user OUs to model real access control (shares, printers, etc.).
- Dedicated Admin Accounts and Service Accounts OUs to support least privilege and auditing.
