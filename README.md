# osTicket — Installation on Azure (Windows 11, IIS, PHP & MySQL)

Deployed **osTicket v1.15.8** on a Windows 11 Azure virtual machine using IIS, PHP 7.3.8, MySQL 5.5.62, and HeidiSQL. This walkthrough documents the installation, database connection, configuration, and successful verification.

> **Lab environment:** The recorded software versions are legacy versions used to reproduce this isolated installation. Do not deploy this configuration as a public production help desk. Keep RDP restricted and deallocate/delete the Azure VM when finished.

## Azure resource group and virtual machine

In Azure Portal, create a resource group named `osticket-rg`. Select your subscription and region.

![01 resource group creation](images/01-resource-group-creation.png)

Open the resource group and confirm that it exists.

![02 resource group overview](images/02-resource-group-overview.png)

Create a Windows 11 Pro virtual machine named `osticket-vm` in East US 2. Select the 4-vCPU, 8-GB size shown and configure the availability and security options.

![03a azure vm instance details](images/03a-azure-vm-instance-details.png)

Create the local administrator account and select Remote Desktop (RDP) access for the lab.

![03b azure vm admin rdp settings](images/03b-azure-vm-admin-rdp-settings.png)

Open **Review + create** and inspect the instance settings and estimated price.

![04a azure vm deployment summary](images/04a-azure-vm-deployment-summary.png)

Review disk and network settings, including the virtual network, subnet, and public IP.

![04b azure vm storage network review](images/04b-azure-vm-storage-network-review.png)

Review the management and monitoring settings before deployment.

![04c azure vm management review](images/04c-azure-vm-management-review.png)

Select **Create** and verify that Azure reports successful deployment.

![05 azure vm deployment success](images/05-azure-vm-deployment-success.png)

Open the deployed VM overview and verify the running status and network configuration.

![06a azure vm overview network](images/06a-azure-vm-overview-network.png)

Check the virtual machine specifications and operating system details.

![06b azure vm system properties](images/06b-azure-vm-system-properties.png)

## Connect to the Windows VM

On the local Windows computer, open **Remote Desktop Connection** and enter the Azure VM public IP.

![07 rdp connection](images/07-rdp-connection.png)

Authenticate using the VM local administrator account created in Azure. Do not save or publish the password.

![08 rdp local admin authentication](images/08-rdp-local-admin-authentication.png)

Confirm that the remote Windows 11 desktop loads.

![09 azure vm rdp success](images/09-azure-vm-rdp-success.png)

## Prepare the installation files and IIS

Download the osTicket lab installation archive onto the VM.

![10 osticket install files download](images/10-osticket-install-files-download.png)

Extract the archive and verify that the installers and osTicket ZIP are present.

![11 osticket install files extracted](images/11-osticket-install-files-extracted.png)

Open **Turn Windows features on or off** and enable **Internet Information Services**, including **World Wide Web Services → Application Development Features → CGI**.

![12 iis cgi feature config](images/12-iis-cgi-feature-config.png)

Open `http://127.0.0.1` and verify that the default IIS welcome page loads.

![13 iis default website verification](images/13-iis-default-website-verification.png)

Install **PHP Manager for IIS** using the included installer.

![14 php manager install complete](images/14-php-manager-install-complete.png)

Install **IIS URL Rewrite Module 2**.

![15 iis url rewrite install complete](images/15-iis-url-rewrite-install-complete.png)

Create the `C:\PHP` folder on the VM.

![16 php install directory](images/16-php-install-directory.png)

Extract the provided PHP 7.3.8 files into `C:\PHP`.

![17 php 7.3.8 files extracted](images/17-php-7.3.8-files-extracted.png)

Install the Microsoft Visual C++ x86 Redistributable included with the lab.

![18 vc redist install complete](images/18-vc-redist-install-complete.png)

## Install MySQL and register PHP

Launch the MySQL Server 5.5 setup wizard.

![19 mysql server setup wizard](images/19-mysql-server-setup-wizard.png)

Choose **Typical** installation and complete the installer.

![20 mysql typical setup](images/20-mysql-typical-setup.png)

In the MySQL configuration wizard, choose **Standard Configuration**.

![21 mysql standard config](images/21-mysql-standard-config.png)

Set a strong MySQL `root` password. Leave remote root access and anonymous accounts disabled.

![22 mysql root account config](images/22-mysql-root-account-config.png)

Execute the configuration and verify that the MySQL service starts successfully.

![23 mysql server config complete](images/23-mysql-server-config-complete.png)

Open **IIS Manager → PHP Manager**. It initially reports that PHP is not registered.

![24 php manager before registration](images/24-php-manager-before-registration.png)

Select **Register new PHP version** and point it to `C:\PHP\php-cgi.exe`.

![25 php cgi exe selection](images/25-php-cgi-exe-selection.png)

Confirm that IIS PHP Manager recognizes PHP 7.3.8.

![26 php 7.3.8 registration complete](images/26-php-7.3.8-registration-complete.png)

## Deploy osTicket to IIS

Extract `osTicket-v1.15.8.zip` and locate the `upload` folder.

![27 osticket extracted upload folder](images/27-osticket-extracted-upload-folder.png)

Move the `upload` folder into `C:\inetpub\wwwroot`.

![28 upload folder copied wwwroot](images/28-upload-folder-copied-wwwroot.png)

Rename that folder to `osTicket` (without spaces). The resulting path is `C:\inetpub\wwwroot\osTicket`.

![29 upload folder renamed osticket](images/29-upload-folder-renamed-osticket.png)

In IIS Manager, expand **Default Web Site** and select the new `osTicket` directory.

![30 osticket directory iis](images/30-osticket-directory-iis.png)

Open `http://localhost/osTicket/setup/` and inspect the installer prerequisites.

![31 iis manager and osticket install](images/31-iis-manager-and-osticket-install.png)

In **PHP Manager → PHP Extensions**, identify the initially disabled extensions.

![32 php extensions before enabling](images/32-php-extensions-before-enabling.png)

Enable `php_imap.dll`, `php_intl.dll`, and `php_opcache.dll` in PHP Manager.

![33 php ext enabled](images/33-php-ext-enabled.png)

Refresh the installer. Confirm the required checks pass; IMAP and Intl are recognized. Zend OPcache and APCu may remain marked as unavailable because they are optional for this lab.

![34 osticket reqs verified](images/34-osticket-reqs-verified.png)

## Prepare the configuration file

In `C:\inetpub\wwwroot\osTicket\include`, locate `ost-sampleconfig.php`.

![35 osticket sample config file](images/35-osticket-sample-config-file.png)

Rename the file to `ost-config.php`.

![36 osticket config file rename](images/36-osticket-config-file-rename.png)

Open the file’s **Properties → Security → Advanced** and disable inheritance as shown.

![37 osticket config inheritance disabled](images/37-osticket-config-inheritance-disabled.png)

For the isolated lab installation, temporarily grant `Everyone` **Full Control** so the installer can write the file. Remove broad write access immediately after installation.

![38 osticket config temp perm](images/38-osticket-config-temp-perm.png)

## Configure and install the application

Select **Continue** in the osTicket installer. Enter the help desk name, administrator information, and a unique administrator password.

![39 osticket install form](images/39-osticket-install-form.png)

Install HeidiSQL from the supplied installation files.

![40 heidisql install complete](images/40-heidisql-install-complete.png)

Open HeidiSQL and connect over **MariaDB or MySQL (TCP/IP)** to `127.0.0.1`, port `3306`, with MySQL `root` credentials.

![41 heidisql mysql connect settings](images/41-heidisql-mysql-connect-settings.png)

Right-click the server, choose **Create new → Database**, and name it `osTicket` (shown using the server’s default collation).

![42 heidisql create osticket database](images/42-heidisql-create-osticket-database.png)

Verify the empty `osticket` database appears in HeidiSQL.

![43 osticket database created](images/43-osticket-database-created.png)

Return to the osTicket installer. Use table prefix `ost_`, hostname `localhost`, database `osticket`, MySQL username `root`, and the database password. Keep credentials out of the public screenshots.

![44 osticket mysql connection settings](images/44-osticket-mysql-connection-settings.png)

Click **Install Now** and confirm the **Congratulations!** installation-success page appears.

![45 osticket install success](images/45-osticket-install-success.png)

## Post-installation cleanup and verification

Delete `C:\inetpub\wwwroot\osTicket\setup` after installation.

![46 osticket setup folder removed](images/46-osticket-setup-folder-removed.png)

Return to the security settings for `include\ost-config.php` and remove temporary write/Full Control access. In this lab the `Everyone` entry was changed to **Read & Execute**; production deployments should limit file access to authorized identities.

![47 osticket config read only permissions](images/47-osticket-config-read-only-permissions.png)

Open `http://localhost/osTicket/scp/`, log in as the administrator, and verify the Staff Control Panel and initial “osTicket Installed!” ticket load correctly.

![48 osticket final verification](images/48-osticket-final-verification.png)

---

**Result:** osTicket installed successfully, the administrator Staff Control Panel loaded, and the application remained accessible after removing the installer folder and restricting the configuration file permissions.
