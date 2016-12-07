#Minimalitstic Certificates And CAs HowTo Using libnss3-tools (certutil)

## Create a CA
Create the certificate database for the CA:

    mkdir CA_db
    certutil -N -d CA_db

Create the CA certificate for this CA:

    certutil -S -d CA_db -n "CA Name" -s "CN=CA Name,O=Company,OU=Department,L=City,ST=State,C=US" -t "CT,," -x -2

Now extract the certificate. It will be needed to create a certificate signing request.

    certutil -L -d CA_db -n "CA Name" -a -o caname.crt

## Create Certificate Signing Request (CSR)
First create a certificate database for your application(s):

    mkdir cert_db
	certutil -N -d cert_db

Link with your CA (import CA certificate):

    certutil -A -d cert_db -n "CA Name" -t "TC,," -a -i caname.crt

Create the CSR:

    certutil -R -d cert_db -s "CN=Nick,O=Company,OU=Department,L=City,ST=State,C=US" -a -o server.req

## Signing CSRs
Sign a CSR to create a valid certificate issued by this CA. The -v parameter defines the validity period in months.

    certutil -C -d CA_db -c "CA Name" -a -i server.req -o server.crt -2 -6 -v 36

## Import Signed Certificate
Import signed certificate into your certificate database:

    certutil -A -d cert_db -n "Nick" -a -i server.crt -t ",,"

Check. List certificates:

    certutil -L -d cert_db

## Other stuff

Delete a certificate from a database

    certutil -D -d cert_db -n "Nick"
