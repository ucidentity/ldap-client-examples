**Requirements:**

- This example connects to LDAP securely using LDAPS.  Please ensure your `php.ini` `openssl.cafile` directive points to a valid and trusted certificate store
- Ensure that you have the PHP LDAP extension installed and configured on your server
- Note that the example below uses paged controls to handle pagination in LDAP queries. The control's oid for paged results is 1.2.840.113556.1.4.319.
- The paged search uses a "cookie" to manage state across pages. This cookie is updated with each iteration to indicate the next page to retrieve.

```php
<?php
// LDAP connection settings
$ldap_server = "ldaps://ldap-test.berkeley.edu:636";
$ldap_bind_user = "uid=ldap_bind,ou=applications,dc=berkeley,dc=edu";
$ldap_bind_password = "*******";

// Connect to the LDAPS server
$ldap_connection = ldap_connect($ldap_server);

// Enable TLS
ldap_set_option($ldap_connection, LDAP_OPT_PROTOCOL_VERSION, 3);
ldap_set_option($ldap_connection, LDAP_OPT_REFERRALS, 0);
// ldap_start_tls only required if using ldap://ldap-test.berkeley.edu:389
//ldap_start_tls($ldap_connection);

// Attempt to bind to the server
if (ldap_bind($ldap_connection, $ldap_bind_user, $ldap_bind_password)) {
    echo "Successfully connected to LDAPS server.\n";

    // LDAP search settings
    $page_size = 999; 
    $base_dn = "dc=berkeley,dc=edu";
    $filter = "(sn=Taylor)";
    $attributes = array("cn", "mail");

    // Initialize the control for paged results
    $control = array(
        'oid' => '1.2.840.113556.1.4.319',  // Object Identifier for paged results
        'iscritical' => true,
        'value' => array('size' => $page_size, 'cookie' => '')  // Page size and cookie
    );

    $cookie = '';  // Paged search cookie (empty for the first request)

    // Loop through paged results until the cookie is empty (no more pages)
    do {
        ldap_set_option($ldap_connection, LDAP_OPT_SERVER_CONTROLS, array($control));

        // Perform LDAP search with paged control
        $search_result = ldap_search($ldap_connection, $base_dn, $filter, $attributes);
        $entries = ldap_get_entries($ldap_connection, $search_result);

        // Display the results for each page
        if ($entries["count"] > 0) {
            for ($i = 0; $i < $entries["count"]; $i++) {
                echo "Common Name: " . $entries[$i]["cn"][0] . "\n";
            }
        }

        // Retrieve the control response and update the cookie
        $server_controls = ldap_parse_result($ldap_connection, $search_result, $errcode, $errmsg, $matcheddn, $referrals, $controls);
        if (is_array($controls)) {
            foreach ($controls as $control) {
                if ($control['oid'] == '1.2.840.113556.1.4.319') {
                    $cookie = $control['value']['cookie'];  // Update the cookie for the next page
                }
            }
        }

    } while ($cookie !== '');  // Continue until the cookie is empty

    // Unbind and close the LDAP connection
    ldap_unbind($ldap_connection);
} else {
    echo "Failed to connect to LDAPS server.\n";
}

?>
```