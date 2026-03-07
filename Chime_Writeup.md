Link to Vuln: https://www.chime.com/wp-includes/wlwmanifest.xml
Below is the code.
<?xml version="1.0" encoding="utf-8" ?>

<manifest xmlns="http://schemas.microsoft.com/wlw/manifest/weblog">

  <options>
    <clientType>WordPress</clientType>
	<supportsKeywords>Yes</supportsKeywords>
	<supportsGetTags>Yes</supportsGetTags>
  </options>

  <weblog>
    <serviceName>WordPress</serviceName>
    <imageUrl>images/wlw/wp-icon.png</imageUrl>
    <watermarkImageUrl>images/wlw/wp-watermark.png</watermarkImageUrl>
    <homepageLinkText>View site</homepageLinkText>
    <adminLinkText>Dashboard</adminLinkText>
    <adminUrl>
      <![CDATA[
			{blog-postapi-url}/../wp-admin/
		]]>
    </adminUrl>
    <postEditingUrl>
      <![CDATA[
			{blog-postapi-url}/../wp-admin/post.php?action=edit&post={post-id}
		]]>
    </postEditingUrl>
  </weblog>

  <buttons>
    <button>
      <id>0</id>
      <text>Manage Comments</text>
      <imageUrl>images/wlw/wp-comments.png</imageUrl>
      <clickUrl>
        <![CDATA[
				{blog-postapi-url}/../wp-admin/edit-comments.php
			]]>
      </clickUrl>
    </button>

  </buttons>

</manifest>


This is an XML file that defines a manifest for Windows Live Writer. It is used to specify settings and options for a blog in the Writer application, such as the blogging platform type, services, image URLs, and buttons. The manifest file is used to provide Writer with information about the blog's capabilities and how to interact with the blog platform.
This code is an XML file that defines a manifest for Windows Live Writer (WLW), a desktop application used for publishing content to blogs. The manifest provides metadata about the blog platform, such as the name and image URL, as well as details about specific features supported by the platform, such as keyword and tag support. The manifest also includes information about buttons that can be added to the WLW user interface for specific actions, such as managing comments.

The XML code is not meant to be accessed by users directly, but rather read and interpreted by the WLW application. This is why it should not be publicly accessible. If the code were to be accessed by an unauthorized user, it could potentially reveal sensitive information about the blog platform, such as login credentials or details about the server configuration. Additionally, if an attacker were able to modify the XML code, they could potentially inject malicious code into the WLW application, compromising the security of the blog platform. Therefore, it is important to keep this code secure and not publicly accessible.

An attacker could use the information contained in this code to potentially gain unauthorized access to a WordPress site.

For example, the URLs contained in this code can be used to access certain pages of a WordPress site, such as the dashboard or the edit post page, without proper authorization. An attacker who gains access to the dashboard could potentially install malware, add malicious content, or perform other nefarious actions.

In addition, the clientType, supportsKeywords, and supportsGetTags options could provide information about how the site is set up and could potentially be exploited to target the site for attacks.

Finally, the image URLs contained in the code could potentially be used to gain further information about the site or to potentially launch a phishing attack.

Overall, while this code may seem harmless on the surface, it could be used by attackers to gain unauthorized access to a WordPress site and potentially cause significant harm. Therefore, it is important that this code not be publicly accessible and that site owners take steps to ensure the security of their WordPress installations.

Assuming that you want to make the code inaccessible from the public web, you can move the file outside of the web root directory, or add an .htaccess file to deny access to the file. You can also use server-side languages like PHP to check if the request is coming from a specific IP address or if the user is authenticated before allowing access to the file.

Here's an example PHP code that you can use to check if the request is coming from an authorized IP address before executing the code and prevent the file from being directly accessed via  URL: https://www.chime.com/wp-includes/wlwmanifest.xml

The Fix.
<?php
// List of authorized IP addresses
$allowed_ips = array('127.0.0.1');

// Check if the request is coming from an authorized IP address
if (!in_array($_SERVER['REMOTE_ADDR'], $allowed_ips)) {
    // Access denied
    header('HTTP/1.0 403 Forbidden');
    exit('Access denied');
}

// The rest of your code here
?>

This code checks if the IP address of the user making the request is in the list of authorized IP addresses. If it is not, a 403 Forbidden HTTP status code is returned, and the code execution is terminated. If the IP address is authorized, the rest of your code can execute.

Note that this is just an example, and you should modify it to suit your needs. For example, you may want to use a more secure way of storing the list of authorized IP addresses, like in a database or a configuration file that is not accessible from the public web.

Writeup by Taylor Christian Newsome Twitter.com/ClumsyLulz
