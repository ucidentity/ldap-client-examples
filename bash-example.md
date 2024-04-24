```bash
# example secure connection using LDAPS
ldapsearch -H ldaps://ldap.berkeley.edu \
-D "$LDAP_USER" \
-w $LDAP_PWD \
-b "dc=berkeley,dc=edu" \
-s sub \
"(uid=2)"

# example secure connection using LDAP and StartTLS
ldapsearch -ZZ -H ldap://ldap.berkeley.edu \
-D "$LDAP_USER" \
-w $LDAP_PWD \
-b "dc=berkeley,dc=edu" \
-s sub \
"(uid=2)"
```