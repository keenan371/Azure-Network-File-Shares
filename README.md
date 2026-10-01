# Network File Shares and Permissions on Windows Server in Azure
### Share permissions, security groups and permission inheritance (Lab 7)

## Project summary

I built four shared folders on a Windows Server domain controller in Azure, gave each one a different level of access, and then tested every one from a second machine as a normal domain user. The key moment: a user was denied the `accounting` share, I added them to an `ACCOUNTANTS` security group, they signed in again, and the same share opened. That one test shows how group based access control works in a real Windows domain. A short second part repeats the idea with Google Drive sharing and folder inheritance.

- **What this is:** a hands-on walkthrough of Windows network file share permissions, tested end to end with screenshots.
- **Languages:** none. Everything was done through the Windows GUI (File Explorer, Active Directory Users and Computers).
- **Environment:** Microsoft Azure virtual machines (Windows Server `DC-1` as the domain controller, Windows `Client-1` as a workstation) from my earlier Active Directory and DNS labs, connected over Remote Desktop. The domain is `mydomain.com`. Part 2 used Google Drive in a browser.
- **Technologies:** Active Directory Domain Services, Active Directory Users and Computers, SMB network shares, share permissions, security groups, Google Drive sharing.

**Note on accuracy:** I only configured **share permissions** in this lab and observed that they controlled the outcome. I did not change NTFS (file system) permissions here, so I do not claim to have. NTFS plus share permissions together is listed under next steps.

## What I tested (the layout)

| Folder on DC-1 | Share permission I set | What a normal user should get |
|---|---|---|
| `read-access` | Domain Users: Read | Can open and read, cannot create files |
| `write-access` | Domain Users: Change (read/write) | Can open and create files |
| `no-access` | Domain Admins: Full Control (Everyone removed) | Cannot even open it |
| `accounting` | `ACCOUNTANTS` group: Change (set in Part 1B) | Denied until the user is in the group |

For every share I removed the default **Everyone** entry first, so access comes only from what I deliberately granted.

---

## Part 1A. Create the shares and test them as a normal user

**Setup.** I turned on the `DC-1` and `Client-1` virtual machines in the Azure Portal, connected to `DC-1` over Remote Desktop as the domain admin (`mydomain.com\jane_admin`), and connected to `Client-1` as a normal domain user (`hhippoford`, one of the accounts from my Active Directory lab).

**Step 1. Create four folders on DC-1.**
On `DC-1`, open File Explorer, go to `C:\`, right-click > New > Folder. Create `read-access`, `write-access`, `no-access` and `accounting`.
*Expected:* four new folders in the `C:\` listing.

![Four folders created](evidence/01-dc1-4-folders-created.png)

**Step 2. Share `read-access` with Read only.**
Right-click `read-access` > Properties > Sharing > **Advanced Sharing** > tick "Share this folder" > **Permissions**. Remove **Everyone**. Click **Add**, type `Domain Users`, OK. Under "Permissions for Domain Users" tick **Read** only.
*Expected:* only Domain Users listed, with Read allowed and Full Control and Change unticked.

![read-access share permissions](evidence/02-dc1-read-access-share-perms-domainusers-read.png)

**Step 3. Share `write-access` with Change (read/write).**
Same steps, folder `write-access`, group `Domain Users`, tick **Change**. (Windows ticks Read automatically.)
*Expected:* Domain Users with Change and Read allowed.

![write-access share permissions](evidence/03-dc1-write-access-share-perms-domainusers-change.png)

**Step 4. Share `no-access` for admins only.**
Same steps, folder `no-access`, group `Domain Admins`, tick **Full Control**. Remove Everyone. Normal users are not listed at all.
*Expected:* only Domain Admins listed.

![no-access share permissions](evidence/04-dc1-no-access-share-perms-domainadmins-only.png)

**Step 5. Skip `accounting` for now** (that is the lab's later test). Confirm the three shares are configured.

![All three shares configured](evidence/05-dc1-all-3-shares-configured.png)

**Step 6. Browse the shares from Client-1 as a normal user.**
On `Client-1`, signed in as `hhippoford`, press Start > Run, type `\\dc-1` and press Enter.
*Expected:* the shares on `DC-1` appear (`read-access`, `write-access`, `no-access`, plus the built in `NETLOGON` and `SYSVOL`). Seeing a share in the list does not mean you can open it.

![Shares visible from Client-1](evidence/06-client1-hhippoford-dc1-shares-visible.png)

**Step 7. Try `read-access`: opening works, creating a file fails.**
Open `read-access`, then try to create a new text file.
*Expected and observed:* the folder opens, but creating a file gives "Destination Folder Access Denied. You need permission to perform this action."

![read-access: write denied](evidence/07-client1-readaccess-write-denied.png)

**Step 8. Try `write-access`: creating a file succeeds.**
Open `write-access` and create a new text document.
*Expected and observed:* the file `New Text Document` is created.

![write-access: file created](evidence/08-client1-writeaccess-file-created-succeeds.png)

**Step 9. Try `no-access`: denied even to open.**
Double-click `no-access`.
*Expected and observed:* "Windows cannot access \\dc-1\no-access. You do not have permission to access \\dc-1\no-access. Contact your network administrator to request access."

![no-access: denied](evidence/09-client1-noaccess-denied.png)

**What this showed:** the three outcomes (read only, read/write, nothing) match exactly what the share permissions said. There were no surprises from the file system underneath.

---

## Part 1B. Group based access with an ACCOUNTANTS security group

**Step 10. Create the group.**
On `DC-1`, open **Active Directory Users and Computers**, right-click the `_EMPLOYEES` OU > New > **Group**. Name `ACCOUNTANTS`, scope **Global**, type **Security**, OK.

![New group dialog](evidence/10-dc1-accountants-group-dialog.png)
![ACCOUNTANTS group created](evidence/11-dc1-accountants-group-created.png)

**Step 11. Share `accounting` to the group only.**
Right-click `accounting` > Properties > Sharing > Advanced Sharing > Permissions. Remove **Everyone**, add `ACCOUNTANTS`, tick **Change**.
*Expected:* only ACCOUNTANTS listed, with Change and Read.

![accounting share permissions](evidence/12-dc1-accounting-share-perms-accountants-change.png)

**Step 12. Test as a user who is NOT in the group.**
On `Client-1`, as `hhippoford` (member of Domain Users only), open `\\dc-1\accounting`.
*Expected and observed:* "Windows cannot access \\dc-1\accounting. You do not have permission."

![accounting denied before group membership](evidence/13-client1-accounting-denied-before-group.png)

**Step 13. Sign the user out of Client-1.**
Start menu > account icon > **Sign out**. This step matters, see the lesson below. (I did not capture a clean screenshot of the sign out, so I am documenting it in text.)

**Step 14. Add the user to ACCOUNTANTS.**
On `DC-1` in Active Directory Users and Computers, open `hhippoford` > **Member Of** tab > **Add** > `ACCOUNTANTS`.
*Expected:* the Member Of list shows `ACCOUNTANTS` and `Domain Users`.

![hhippoford added to ACCOUNTANTS](evidence/14-dc1-hhippoford-added-to-accountants.png)

**Step 15. Sign back in and test again.**
Sign `hhippoford` back into `Client-1`, open `\\dc-1\accounting`.
*Expected and observed:* the share opens (empty folder, no error). Same user, same share, different result, because of one group membership.

![accounting access succeeds after joining the group](evidence/15-client1-accounting-access-succeeds-after-group.png)

### Lesson: sign out and back in
Group membership did not take effect while `hhippoford` stayed signed in. Windows puts a user's group memberships into their Kerberos ticket at sign in, so a change only shows up on a fresh sign in. This is the same reason removing someone from a group does not instantly cut off a session they already have open.

### Other things I hit
- Each VM allows only one Remote Desktop session. Opening the connection again kicks the first session and needs the password again.
- I tested with a normal domain user on purpose. As a domain admin everything would have opened and proven nothing.

---

## Part 2. Google Drive permissions and inheritance (no Azure)

I repeated the same ideas in Google Drive using my own account and the browser. **What I actually did live:**
- Created a document, "Lab7 Sharing Test Doc", containing one word.
- Set General access to **Anyone with the link: Viewer**, then **Anyone with the link: Editor**, then back to **Restricted** (owner only).
- Created a folder named **Public Documents** and set it to **Anyone with the link: Viewer**.
- Moved the restricted document into that folder. Google showed the warning "Change who has access? This item will be visible to everyone who can see 'Public Documents'."
- Reopened the document's Share dialog afterwards and confirmed General access had **automatically changed from Restricted to Anyone with the link: Viewer**. The document inherited the folder's permission without any change on the document itself.

**What I did not do, to be honest about it:** the lab asks you to open the link in a private (incognito) window at several points (steps 3, 5, 6, 8 and 13). The browser tool I used for Part 2 has no private window, so I did not test as a logged out visitor. I described what a logged out visitor would see from Google's documented behavior instead. The inheritance result above was confirmed directly. I also did not save screenshots for Part 2, so there are no images for it here.

---

## What I would do next
- Combine **NTFS permissions with share permissions** and show that the most restrictive of the two wins.
- Use the standard pattern of putting users in **global groups**, groups in **domain local groups**, and granting permissions to the domain local group (AGDLP).
- Turn on **access based enumeration** so users only see the shares they can open.
- Log access attempts and review them, so denied attempts are visible.

## Cleanup
After I saved all the evidence, I deleted the lab's Azure resources so they stop billing.

## Repository layout
```
README.md
evidence/   15 screenshots from the DC-1 and Client-1 virtual machines (lab accounts only)
```
