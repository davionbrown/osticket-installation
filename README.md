# osTicket — Installation on Azure (Windows 11, IIS, PHP & MySQL)

Installed **osTicket v1.15.8** on a Windows 11 virtual machine in Microsoft Azure using **Internet Information Services (IIS)**, **PHP 7.3.8**, **MySQL Server 5.5.62**, and **HeidiSQL 12.3**. This walkthrough records the installation from Azure provisioning through administrator login, with screenshots of each major step.

## Azure resource group and virtual machine

**1. Create a resource group.** In the Azure portal, open **Resource groups → Create**. Select the subscription and **East US 2** region, enter `osticket-rg`, then select **Review + create → Create**.

![Create Azure resource group](images/01-resource-group-creation.png)

**2. Verify the resource group.** Open `osticket-rg` and confirm that it appears in the Azure portal.

![Azure resource group overview](images/02-resource-group-overview.png)

**3. Configure the virtual machine.** Open **Virtual machines → Create → Azure virtual machine**. Choose resource group `osticket-rg`, enter `osticket-vm`, select **Windows 11 Pro** as the image, and use the demonstrated **4-vCPU, 8-GiB** VM size. Set the availability and security options shown in the deployment.

![Azure VM instance details](images/03a-azure-vm-instance-details.png)

**4. Configure administrator and RDP access.** Create a local administrator account and a unique password. Under inbound ports, select **RDP (3389)** for the temporary lab connection.

![Azure VM administrator and RDP settings](images/03b-azure-vm-admin-rdp-settings.png)

**5. Review the VM configuration.** Select **Review + create**. Confirm the VM name, Windows image, region, instance size, and estimated cost.

![Azure VM deployment summary](images/04a-azure-vm-deployment-summary.png)

**6. Review storage and networking.** Verify the OS disk, virtual network, subnet, network security settings, and public IP configuration.

![Azure VM storage and network review](images/04b-azure-vm-storage-network-review.png)

**7. Review management options.** Confirm the selected management and monitoring settings before deploying the VM.

![Azure VM management review](images/04c-azure-vm-management-review.png)

**8. Deploy the VM.** Select **Create**, wait for deployment to finish, and verify that Azure reports **Your deployment is complete**.

![Azure VM deployment successful](images/05-azure-vm-deployment-success.png)

**9. Check the VM overview.** Open `osticket-vm` and verify the VM is running. Locate its public IP address for the RDP connection.

![Azure VM overview and network](images/06a-azure-vm-overview-network.png)

**10. Confirm system details.** Review the VM's operating system and hardware specifications.

![Azure VM system properties](images/06b-azure-vm-system-properties.png)

## Connect to the Windows VM

**11. Start Remote Desktop.** On a Windows computer, open **Remote Desktop Connection** (`mstsc`) and enter the VM's public IP address.

![Remote Desktop connection](images/07-rdp-connection.png)

**12. Authenticate.** Enter the local administrator username and password created during VM deployment. Do not include credentials in screenshots or the repository.

![Remote Desktop administrator authentication](images/08-rdp-local-admin-authentication.png)

**13. Verify the session.** Confirm that the remote Windows 11 desktop loads.

![Successful RDP session](images/09-azure-vm-rdp-success.png)

## Prepare the installation files and IIS

**14. Download the installation archive.** On the VM, download the installation bundle used for this lab, `osTicket-Installation-Files.zip`.

![Download osTicket installation files](images/10-osticket-install-files-download.png)

**15. Extract the archive.** Right-click the ZIP file, choose **Extract All**, and verify that the folder contains the osTicket ZIP and installers for PHP Manager, IIS URL Rewrite, PHP, Visual C++, MySQL, and HeidiSQL.

![Extracted installation files](images/11-osticket-install-files-extracted.png)

**16. Enable IIS and CGI.** Open **Control Panel → Programs → Turn Windows features on or off**. Enable **Internet Information Services** and navigate to **World Wide Web Services → Application Development Features → CGI**. Apply the changes.

![Enable IIS and CGI](images/12-iis-cgi-feature-config.png)

**17. Test IIS.** In a browser on the VM, open `http://127.0.0.1` and verify that the default IIS welcome page appears.

![IIS default website verification](images/13-iis-default-website-verification.png)

**18. Install PHP Manager.** Run the included **PHP Manager for IIS** installer and complete the setup wizard.

![PHP Manager installation complete](images/14-php-manager-install-complete.png)

**19. Install IIS URL Rewrite.** Run the included **IIS URL Rewrite Module 2** installer and finish setup.

![IIS URL Rewrite installation complete](images/15-iis-url-rewrite-install-complete.png)

**20. Create the PHP directory.** In File Explorer, navigate to `C:\` and create a folder named `PHP`.

![Create PHP installation directory](images/16-php-install-directory.png)

**21. Extract PHP.** Extract the **PHP 7.3.8 non-thread-safe x86** archive into `C:\PHP`. Verify that the PHP files, including `php-cgi.exe`, are directly inside that directory.

![PHP 7.3.8 files extracted](images/17-php-7.3.8-files-extracted.png)

**22. Install the Visual C++ runtime.** Run the included **Microsoft Visual C++ x86 Redistributable** installer and complete installation.

![Visual C++ Redistributable installation complete](images/18-vc-redist-install-complete.png)

## Install MySQL and register PHP

**23. Launch MySQL setup.** Open the **MySQL Server 5.5.62** installer from the extracted installation files.

![MySQL setup wizard](images/19-mysql-server-setup-wizard.png)

**24. Select the installation type.** Choose **Typical**, continue through the installer, and finish installing MySQL Server.

![MySQL Typical installation](images/20-mysql-typical-setup.png)

**25. Configure the MySQL instance.** In the MySQL Server Instance Configuration Wizard, select **Standard Configuration**.

![MySQL Standard Configuration](images/21-mysql-standard-config.png)

**26. Set database security options.** Enter and confirm a unique MySQL `root` password. Leave **remote root access** and **anonymous accounts** disabled.

![MySQL root account configuration](images/22-mysql-root-account-config.png)

**27. Start MySQL.** Execute the configuration. Confirm the wizard successfully prepares the configuration, writes the configuration file, starts the service, and applies the security settings.

![MySQL server configuration completed](images/23-mysql-server-config-complete.png)

**28. Inspect PHP Manager.** Open **Internet Information Services (IIS) Manager**, select the server, and open **PHP Manager**. Before registration, PHP is reported as unavailable.

![PHP Manager before registration](images/24-php-manager-before-registration.png)

**29. Register PHP.** Select **Register new PHP version** and enter the executable path `C:\PHP\php-cgi.exe`. Confirm the selection.

![Select PHP CGI executable](images/25-php-cgi-exe-selection.png)

**30. Verify registration.** Confirm that PHP Manager displays **PHP 7.3.8** and the `C:\PHP\php-cgi.exe` executable. Restart IIS to apply the changes.

![PHP registration complete](images/26-php-7.3.8-registration-complete.png)

## Deploy osTicket to IIS

**31. Extract osTicket.** Extract `osTicket-v1.15.8.zip` and locate the `upload` directory inside the extracted folder.

![osTicket extracted upload folder](images/27-osticket-extracted-upload-folder.png)

**32. Move the application files.** Move—not copy—the `upload` directory into the IIS website root: `C:\inetpub\wwwroot`.

![Move upload folder into wwwroot](images/28-upload-folder-copied-wwwroot.png)

**33. Rename the folder.** Rename `upload` to `osTicket`. The final application path is `C:\inetpub\wwwroot\osTicket`.

![Rename upload folder to osTicket](images/29-upload-folder-renamed-osticket.png)

**34. Locate osTicket in IIS.** In IIS Manager, expand **Sites → Default Web Site** and select the `osTicket` directory. Use **Browse *:80 (http)** to open the site.

![osTicket directory in IIS Manager](images/30-osticket-directory-iis.png)

**35. Inspect the installer prerequisites.** Open `http://localhost/osTicket/setup/`. Review the required and recommended PHP extension checks before continuing.

![osTicket installer prerequisites](images/31-iis-manager-and-osticket-install.png)

**36. Inspect PHP extensions.** In IIS Manager, open **PHP Manager → Enable or disable an extension** and locate the disabled extensions needed by the installer.

![PHP extensions before enabling](images/32-php-extensions-before-enabling.png)

**37. Enable the extensions.** Enable `php_imap.dll`, `php_intl.dll`, and `php_opcache.dll` in PHP Manager.

![PHP extensions enabled](images/33-php-ext-enabled.png)

**38. Recheck prerequisites.** Refresh the osTicket installer. Verify that the required checks pass and that the IMAP and Intl checks are green. In this lab, the installer still marked **Zend OPcache** and **APCu** as unavailable; these were recommended rather than required for completing the installation.

![osTicket requirements verified](images/34-osticket-reqs-verified.png)

## Prepare the configuration file

**39. Find the sample configuration file.** Open File Explorer and navigate to `C:\inetpub\wwwroot\osTicket\include`. Locate `ost-sampleconfig.php`.

![Locate osTicket sample configuration file](images/35-osticket-sample-config-file.png)

**40. Rename the file.** Rename `ost-sampleconfig.php` to `ost-config.php`, preserving the `.php` extension.

![Rename osTicket configuration file](images/36-osticket-config-file-rename.png)

**41. Disable inherited permissions for the installation.** Right-click `ost-config.php` and select **Properties → Security → Advanced**. Select **Disable inheritance**, then choose **Remove all inherited permissions from this object**. Confirm that the permissions list is empty and the button changes to **Enable inheritance**. The file owner can still adjust permissions.

![Disable inherited permissions on ost-config.php](images/37-osticket-config-inheritance-disabled.png)

**42. Grant temporary installer access.** In **Advanced Security Settings**, select **Add → Select a principal**. Enter `Everyone`, choose **Check Names → OK**, set **Type: Allow** and **Basic permissions: Full control**, then select **OK → Apply**. This temporary, broad permission was used only to allow the installer to write `ost-config.php` in the isolated lab. It must not remain writable after installation.

![Temporary osTicket configuration file permissions](images/38-osticket-config-temp-perm.png)

## Configure and install the application

**43. Open the installation form.** Return to `http://localhost/osTicket/setup/` and select **Continue**. Enter the help desk name and email address, administrator name, email address, username, and a unique administrator password. Leave the database settings for the next steps.

![osTicket basic installation form](images/39-osticket-install-form.png)

**44. Install HeidiSQL.** Run the included **HeidiSQL 12.3** installer, complete the wizard, and launch the application.

![HeidiSQL installation complete](images/40-heidisql-install-complete.png)

**45. Connect to MySQL.** In HeidiSQL Session Manager, select **MariaDB or MySQL (TCP/IP)**. Set **Hostname/IP** to `127.0.0.1`, **User** to `root`, and **Port** to `3306`. Enter the MySQL password created earlier and open the connection.

![HeidiSQL MySQL connection settings](images/41-heidisql-mysql-connect-settings.png)

**46. Create the database.** Right-click the connected server and select **Create new → Database**. Enter `osTicket` as the database name and retain the demonstrated server-default collation, `latin1_swedish_ci`. Select **OK**. The resulting database appears as `osticket` in this installation.

![Create osTicket database in HeidiSQL](images/42-heidisql-create-osticket-database.png)

**47. Verify the empty database.** Select `osticket` in HeidiSQL and confirm it exists before installing the application.

![Verify newly created osTicket database](images/43-osticket-database-created.png)

**48. Complete database settings in the installer.** Return to the osTicket installation form and enter the following values:

- **MySQL Table Prefix:** `ost_`
- **MySQL Hostname:** `localhost`
- **MySQL Database:** `osticket`
- **MySQL Username:** `root`
- **MySQL Password:** the password configured during MySQL setup

The `root` account was used to reproduce this lab. A production installation should instead use a dedicated database user with only the required privileges.

![osTicket database connection settings](images/44-osticket-mysql-connection-settings.png)

**49. Install osTicket.** Select **Install Now**. Wait for the **Congratulations!** page confirming that osTicket installation completed successfully.

![osTicket installation successful](images/45-osticket-install-success.png)

## Post-installation cleanup and verification

**50. Remove the installer directory.** In File Explorer, open `C:\inetpub\wwwroot\osTicket`. Delete the `setup` folder and confirm it is no longer present. This prevents the installer from remaining accessible.

![osTicket setup folder removed](images/46-osticket-setup-folder-removed.png)

**51. Remove temporary write access from the configuration file.** Return to `C:\inetpub\wwwroot\osTicket\include\ost-config.php`, right-click the file, and open **Properties → Security → Advanced**. In the recorded lab, the `Everyone` entry was edited from **Full control** to **Read & execute**, removing write and modification permissions. Select **Apply → OK** to save the change.

![osTicket configuration file read and execute permissions](images/47-osticket-config-read-only-permissions.png)

**52. Verify administrator access.** In the VM browser, open `http://localhost/osTicket/scp/` and sign in using the administrator account created during installation. Confirm that the **Staff Control Panel** opens and the initial **“osTicket Installed!”** ticket appears.

![osTicket administrator dashboard and initial ticket](images/48-osticket-final-verification.png)

---

**Result:** osTicket v1.15.8 installed successfully on the Azure Windows 11 VM. IIS served the application, the MySQL database connection succeeded, the installer directory was removed, temporary configuration-file write access was revoked, and the administrator Staff Control Panel loaded with the initial installation ticket.

