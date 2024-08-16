---
title: PHP Active Directory Search 
description: ""
date: 2008-04-5T06:45:27.051Z
draft: false
tags: []
categories: []
---

Just add the fully qualified domain name of your domain controller (host), the domain dn ( if your domain was beanz.meanz.heinz.com for example your dn would be DC=beanz,DC=meanz,DC=heinz,DC=com ) supply the username and password of a user in your domain, this user does not need to be a member of any group, as long as the account isn’t disabled. Then your off!You can add active directory attributes to the $search_fields array to change what you can search on
You can add which active directory attributes are displayed in the result by adding to the $return_fields array


the key for these array is the Human readable version (i.e. what will be displayed) so if your using custom attributes you can show them as a friendly name.Remember this wont work, with out the php LDAP module enabled, this can be easily done by uncommenting the module in your php.ini

{{ < highlight php > }}
<?php
// CONFIGURATION START
$config = array  ('host'    => 'domaincontroller.domain.com',
     'dn'     => 'DC=domain,DC=com',
     'username'   => 'myusername@domain.com',
      'password'   => 'password',
     'color1'    => '#DDDDDD',
     'color2'    => '#FFFFFF',
     );
// $search_fields ,  select which AD fields you want to search on
$search_fields=array('First Name'   => 'givenname',
      'Last Name'   => 'sn',
      );
//$return_fields , select which AD fields you wish to have displayed in the results
$return_fields=array('Display Name'  => 'displayname',
     'First Name'   => 'givenname',
     'Last Name'   => 'sn',
     'Telephone Number'  => 'telephonenumber',
     'Email Address'  => 'mail',
     'Company'    => 'company',
      );
//CONFIGURATION END
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> <html xmlns=" http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
<title>ACTIVEDIRECTORY SEARCH TOOL </title>
<style type="text/css">
<!--
body,td,th {
    font-family: Verdana, Arial, Helvetica, sans-serif;
    font-size: 10px;
}
-->
</style></head><body>
<form id="form1" name="form1" method="post" action="">
<?php
foreach ($search_fields as $key => $value)
     { ?>
  <label>
<b><?php echo $key; ?></b> <input name="<?php echo $value; ?>" type="text" id="givenname" />
  </label>
<?php } ?>
  <label>
  <input name="Search" type="submit" id="Search" value="Search" />
  </label>
</form>
<?php
$ldap_search='';
if ($_SERVER['REQUEST_METHOD'] == "POST")
{
foreach ($search_fields as $item => $value) //build filter
  {
  if (empty($_POST[$value]))
   {
    $filter[$value]="($value=*)";
   } else {
    $filter[$value]="($value=$_POST[$value])";
   }
  $ldap_search=$ldap_search.$filter[$value];
  }
 $attrs=array();
 foreach ($return_fields as $item => $value) //build attr array
  {
  $attrs[]=$value;
  }
 $ldap_search='(&'.$and.$ldap_search.')';
$ad = ldap_connect($config['host'])
      or die( "Could not connect!" );
// Set version number
ldap_set_option($ad, LDAP_OPT_PROTOCOL_VERSION, 3)
     or die ("Could not set ldap protocol");
ldap_set_option($ad, LDAP_OPT_REFERRALS,0)
 or die ("could no se the ldap referrals");
// Binding to ldap server
$bd = ldap_bind($ad, $config['username'], $config['password'])
      or die ("Could not bind");
// Create the DN
// Specify only those parameters we're interested in displaying
// Create the filter from the search parameters
$search = ldap_search($ad, $config['dn'], $ldap_search, $attrs)
          or die ("ldap search failed");
$entries = ldap_get_entries($ad, $search);
if ($entries["count"] > 0) {
$row_count = 0;
?>
No of results = <?php echo $entries["count"] ?>
</p>
<table width="100%" border="1">
  <tr>
   <?php
 foreach ($return_fields as $item => $value)
  {
   ?><th><?php echo $item ?></th><?php
  }
 ?>
  </tr><? for ($i=0; $i<$entries["count"]; $i++) {
  $row_color = ($row_count % 2) ? $config['color1'] : $config['color2'];
  ?>
  <tr bgcolor="<?php echo $row_color ?>">
    <?php
 foreach ($return_fields as $item => $value)
  {
  if (isset ($entries[$i][$value][0]))
   {
   if ($value == 'mail') // do i need a mailto: link ?
    {
    echo "<td><a href="mailto:" .$entries[$i][$value][0]. "">" .$entries[$i][$value][0]. "</a></td>" ;
    } else {
    echo '<td>'.$entries[$i][$value][0].'</td>';
    }
   } else {
   echo '<td></td>'; //fial not set output empty cell
   }
  }
 ?>
 </tr>
<?
$row_count++;
 }
?>
</table>
<?
} else {
   echo "<p>No results found!</p>";
}
ldap_unbind($ad);
}
?>
<center><small><a href="http://www.james-lloyd.com">script by James Lloyd</a></small></center></html>
{{ < / highlight >}}