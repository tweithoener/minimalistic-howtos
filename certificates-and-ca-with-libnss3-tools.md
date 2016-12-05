
#Create a CA

    mkdir CA_db

Create the certificate database for the CA

    certutil -N -d CA_db

Create the CA certificate for this CA;

    certutil -S -d CA_db -n "CA Name" -s "CN=CA Name,O=Company,OU=Department,L=City,ST=State,C=US" -t "CT,," -x -2

Now extract the certificate, which will be needed to create a certificate signing request

    certutil -L -d CA_db -n "CA Name" -a -o caname.crt

# Create Certificate Signing Request (CSR)

First create a certificate database to use together with the CA

    mkdir cert_db
	certutil -N -d cert_db

Link with your CA (import CA certificate)

    certutil -A -d cert_db -n "CA Name" -t "TC,," -a -i caname.crt

Create the CSR

    certutil -R -d cert_db -s "CN=Usage,O=Company,OU=Department,L=City,ST=State,C=US" -a -o server.req

# Signing

Sign a CSR to create a valid certificate issued by this CA

    certutil -C -d CA_db -c "CA Name" -a -i server.req -o server.crt -2 -6

# Import Signed Certificate

Import signed certificate into your certificate database

    certutil -A -d cert_db -n "Usage" -a -i server.crt -t ",,"

Check 

    certutil -L -d cert_db
