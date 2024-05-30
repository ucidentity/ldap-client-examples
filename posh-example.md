1. Create a script named `ldapSearch.ps1` with the following contents

```powershell
 param (
    [Parameter(Mandatory = $true)]
    [string]$uid
)

[System.Reflection.Assembly]::LoadWithPartialName("System.DirectoryServices.Protocols")

# Get your credential (your LDAP BIND and password)
$userName=Read-Host -Prompt:"Enter username"
$securePwd=Read-Host -AsSecureString -Prompt:"Enter password" 
$cred=new-object System.Net.NetworkCredential($userName,$securePwd) 

# Define the server and port for LDAPS
$ldapServer = "ldap.berkeley.edu"
$ldapPort = 636

# Define the base DN and filter
$baseDN = "ou=people,dc=berkeley,dc=edu"
$filter = "(&(objectClass=person)(uid=$uid))"

# Create an LDAP connection
$ldapConnection = New-Object System.DirectoryServices.Protocols.LdapConnection((new-object System.DirectoryServices.Protocols.LdapDirectoryIdentifier($ldapServer, $ldapPort)), $cred) 
$LdapConnection.AuthType = [System.DirectoryServices.Protocols.AuthType]'Basic'
$ldapConnection.SessionOptions.SecureSocketLayer = $truea
$ldapConnection.SessionOptions.VerifyServerCertificate = { $true }

# Define the search scope and attributes to load
$searchScope = [System.DirectoryServices.Protocols.SearchScope]::Subtree
$attributes = @("uid", "cn", "sn", "mail") # Adjust attributes as needed

# Create the search request
$searchRequest = New-Object System.DirectoryServices.Protocols.SearchRequest $baseDN, $filter, $searchScope
foreach($attribute in $attributes)
{
    $searchRequest.Attributes.Add($attribute) | Out-Null
}

try {
    # Send the search request
    $searchResponse = $ldapConnection.SendRequest($searchRequest)

    # Check if the response contains any entries
    if ($searchResponse.Entries.Count -gt 0) {
        foreach ($entry in $searchResponse.Entries) {
            Write-Host "DN: $($entry.DistinguishedName)"
            foreach ($attribute in $attributes) {
                if ($entry.Attributes[$attribute]) {
                    Write-Host "${attribute}: $($entry.Attributes[$attribute][0])"
                }
            }
            Write-Host ""
        }
    } else {
        Write-Host "No entries found."
    }
} catch {
    Write-Error "An error occurred: $_"
} finally {
    # Dispose of the connection
    $ldapConnection.Dispose()
} 
```

2. Run the script as `.\ldapSearch.ps1 -uid 888236`
3. You will be prompted for your credentials.  The credentials should be your LDAP bind and password (e.g. uid=my-ldap-service,ou=applications...)
4. Edit as necessary.
