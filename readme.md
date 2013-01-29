# Topics:
* Tableau Server Configuration
* Installation
* Configuration
* Embed tableau reports in confluence

## Tableau Server Configuration
* Alter httpd template
By default, tableau runs either in SSPI mode or you can configure trusted authentication for authentication. But what if you want to do both?
In our situation we use a plugin to export reports on confluence pages, so confluence needs to authenticate with the user credentials of the currently logged on user. And normal users should be able to use SSPI.

To enable both features alter the file with filename: httpd.conf.templ

Tableau 6.x

1. Open the template file:

    ```C:\Program Files (x86)\Tableau Server\6.0\templates```

2. Search for entry:

    ```<Location ~ "^/(?!manual|auth).*">```

3. Add 'trusted' to the regular expression:

    ```<Location ~ "^/(?!manual|auth|trusted).*">```

Tableau 7.x

1. Open the template file:

    ```C:\Program Files (x86)\Tableau Server\7.0\templates```

2. Search for the entry:

    ```<Location ~ "^/(?!(t/[^/]*/)?(manual|auth|dataserver)).*">```

3. Add 'trusted' and 'vizql/trusted' to the regular expression:

    ```<Location ~ "^/(?!(t/[^/]*/)?(manual|auth|trusted|vizql/trusted|dataserver)).*">```

### Disable client ip checking
Run following tabadmin commands:
Note tabadmin can be found here: "C:\Program Files (x86)\Tableau\Tableau Server\7.0\bin"

	tabadmin set wg_server.extended_trusted_ip_checking false
	tabadmin configure
	tabadmin restart

### Whitelist your confluence server(s)
Run following tabadmin commands:
    tabadmin set wgtrusted_hosts "192.168.1.1, 192.168.1.2" (where 192.168.1.x are your confluence hosts)
    tabadmin configure
    tabadmin restart

	The comma separated list of ip addresses should be within double quotes with a space after each comma.

	For more information see the tableau server admin reference (page 224 / 225 / 229):
	http://downloads.tableausoftware.com/quickstart/server-guides/en-us/server_admin7.0.pdf

## Installation of plugin
* Get the jar
    * Download the latest jar file from: https://plugins.atlassian.com/plugins/tableau-plugin
	* Or build it from the source : https://github.com/schubergphilis/tableau-confluence-plugin
* Login as a confluence admin
* Goto the menu 'Browse'
* Choose 'Confluence Admin'
* In the section 'Configuration' choose 'Plugins'
* Click on the 'Install' tab
* And choose 'Upload plugin'
* After this process you should see within 'Manage Existing' the 'Schuberg Philis Tableau Plugin'

## Configuration of plugin
Now that you have installed the plugin, configure your tableau hosts

Use a key, for instance: 'prod' or 'production' and a value 'http://tableauserver' or 'https://tableauserver'
It's possible to enter multiple hosts, but the first entry is the default entry.

There is one special entry for debugging purposes, and that is the key 'debugusername', with this entry you can override the username that is used for trusted authentication.

If your tableau instance requires a domain to authenticate you users, you can configure it with the key 'domain', for instance 'mydomain.local'.

## Embed tableau reports in confluence

Possible parameters:
| Parameter   | Default value | Description                           |
| ----------- | ------------- | ------------------------------------- |
| title       |               | name of report, this title is shown when opening the full page interactive view from the 'interactive button' |
| workbook    |               | workbook name on the Tableau server (case sensitive) |
| report      |               | report name in the workbook (case sensitive) |
| height      | 550           | height in number of pixels |
| width       | 1280          | width in number of pixels |
| environment | prod          | option to point to other environment configured in section 'Configuration of plugin', for example use acc for acceptance environment |
| interactive | false         | opens sheet in interactive mode |
| embed       | true          | embed in page |
| toolbar     | true          | show or hide the toolbar |
| tabs        | false         | show tabs for all reports within the workbook |
| borderstyle |               | extra css borderstyle properties, like 'border:red 1px solid' or 'background-color:gray' |
| button      | false         | option to add a button 'Interactive view' to the non interactive (png) view and open the interactive view in a window |
| noprint     | false         | when setting this option to true, this report will be skipped when printing / exporting to pdf or word |
| parameters  |               | send extra parameters to the report |
| refresh     | false         | refresh will invalidate the tableau cached png |
| site        |               | to specify the site to use |

Basically there are two modes
* image view, a plain png type of image
* interactive view, where end users can use parameters, filters, etc provided within the sheet

Examples:

Basic example:

    {tableau-plugin:workbook=TestWorkbook|report=Sheet1}

Set width and height

    {tableau-plugin:workbook=TestWorkbook|report=Sheet1|width=1500|height=800}

Open interactive view and point to acceptance environment

    {tableau-plugin:workbook=TestWorkbook|report=Sheet1|interactive=true|environment=acc}

Example of using parameters

    {tableau-plugin:workbook=TestWorkbook|report=Sheet1|parameters=colour=blue&othervalue=100}
