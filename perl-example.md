```perl
use warnings;
use Net::LDAP;
use Net::LDAP::Control::Paged;

my $ldap_server = "ldaps://ldap.berkeley.edu:636";
my $ldap_base_dn = "dc=berkeley,dc=edu";
my $ldap_user = $ENV{LDAP_USER} // die "Environment variable LDAP_USER not set";;
my $ldap_password = $ENV{LDAP_PWD} // die "Environment variable LDAP_PWD not set";;
my $search_filter = "(&(objectClass=Person)(uid=888*))";
my $page_size = 999;

# Connect to the LDAP server
my $ldap = Net::LDAP->new($ldap_server) or die "Could not connect to $ldap_server: $@";

# Bind to the server
my $mesg = $ldap->bind($ldap_user, password => $ldap_password);
if ($mesg->code) {
    die "Failed to bind: ", $mesg->error;
}

# Paged control
my $page = Net::LDAP::Control::Paged->new(size => $page_size);

# Loop through paged results until the cookie is empty (no more pages)
my $cookie;
while (1) {
    my $result = $ldap->search(
        base    => $ldap_base_dn,
        filter  => $search_filter,
        attrs   => ['uid','berkeleyEduKerberosPrincipalString'],
        control => [$page],
    );

    if ($result->code) {
        die "Search failed: ", $result->error;
    }

    foreach my $entry ($result->entries) {
        print "DN: ", $entry->dn, "\n";
        foreach my $attr ($entry->attributes) {
            print "  $attr: ", join(", ", $entry->get_value($attr)), "\n";
        }
        print "\n";
    }

    # Get paged control cookie
    my ($resp) = $result->control(LDAP_CONTROL_PAGED) or last;
    $cookie = $resp->cookie or last;

    # Set the cookie for the next request
    $page->cookie($cookie);

    # Exit if the cookie is not available
    last if ($cookie eq '');
}

$ldap->unbind;
```