

```python
from ldap3 import Server, Connection, SAFE_SYNC
from ldap3 import Tls
import ssl
import os

USER = 'uid=my_app_bind,ou=applications,dc=berkeley,dc=edu'
PASSWD = os.environ['LDAP_PWD']
HOST = 'ldap.berkeley.edu'
BASE = 'dc=berkeley,dc=edu'

def get_connection():
    server = Server(HOST, use_ssl=True)
    c = Connection(server, USER, PASSWD, auto_bind=True)
    return c

def paged_search(c, filter):
    entries = c.extend.standard.paged_search(BASE, filter, attributes=['uid', 'givenName'], paged_size=99)
    return entries

def main():
    # get a connection
    c = get_connection()

   # perform a paged search when expecting a large number of returned resources
    # (e.g. all members of a department)
    filter = "(departmentNumber=VRSEC)"
    entries = paged_search(c,filter)
    for entry in entries:
       print(f"{entry}")

   # close the connection after all searches are done
    c.unbind()

if __name__ == "__main__":
    main()
```