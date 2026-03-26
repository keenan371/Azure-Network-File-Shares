# Azure Network File Shares and NTFS Permissions

This lab demonstrates the implementation of Network File Shares (SMB) and the configuration of NTFS permissions within a cloud-hosted Windows environment. I focus on the "Principle of Least Privilege" by ensuring users only have access to the data necessary for their roles while strictly denying unauthorized access.

## 📁 Network Share Configuration
In this section, I established the foundation for centralized storage by creating a network share on the file server.

* **SMB Share Setup:** Configuring the sharing properties to make the "Finance" folder accessible over the network.
<img width="1053" alt="Network Share Configuration" src="https://github.com/user-attachments/assets/347526b8-c3a4-430d-b1a0-6a196745cbe6" />

---

## 🔐 NTFS & Security Permissions
Security is applied at the file-system level to ensure that even if a user reaches the folder, their specific actions (Read, Write, Modify) are controlled.

* **Granular Access Control:** Setting up NTFS permissions for specific Security Groups. This ensures that only authorized personnel can modify sensitive company data.
<img width="807" alt="NTFS & Security Permissions" src="https://github.com/user-attachments/assets/1c3b6ae5-4aa9-4e9d-b044-0d551cb67ba5" />

---

## ✅ Implementation & Verification
The final step was verifying that the organizational structure and permissions were correctly applied from a client workstation.

* **Folder Hierarchy:** A clear view of the departmental organization, ensuring a structured and secure environment for end-users.
<img width="1053" alt="Implementation & Verification" src="https://github.com/user-attachments/assets/81b440a7-9dd7-425b-883b-b26e8418c258" />
