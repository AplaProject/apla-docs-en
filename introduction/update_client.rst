######################################################################
Update Management Client 
######################################################################

A REST client that allows to communicate with the update server using command prompt.
Allows for:

* Uploading binaries to the update server
* Downloading binaries from the update server
* Removing binaries from the update server
* Requesting a list of versions of binaries available on the update server
* Generating keys  
* Updating the go-apla binary

***********************************************************************
Location
***********************************************************************
tools/update_client/

***********************************************************************
Commands and Flags
***********************************************************************

-----------------------------------------------------------------------
add-binary
-----------------------------------------------------------------------
Adds a binary to the update server.

* **--server** - address of the update server.
* **--login**  - your login on the update server.
* **--password** - your password on the update server.
* **--binary-path** - path to the binary.
* **--start-block** - the block number from which this binary can be used.
* **--version** - version name of the binary.
* **--key-path** - path to the private key for signature of the binary.

------------------------------------------------------------------------
get-binary
------------------------------------------------------------------------
Download a binary from the update server.

* **--server** - address of the update server. 
* **--version** - binary version to download.
* **--binary-path** - path to the directory to download the binary to.
* **--publ-key-path** - path to the public key.

------------------------------------------------------------------------
remove-binary
------------------------------------------------------------------------
Remove a binary version from the update server.

* **--server** - address of the update server.
* **--login** -  your login on the update server.
* **--password** - your password on the update server.
* **--version**  - binary version to remove.

------------------------------------------------------------------------
generate-keys
------------------------------------------------------------------------
Generate a private-public key pair.

* **--publ-key-path** - path to the public key. By default – resources/key.pub.
* **--key-path**      - path to the private key. By default – resources/key.

-------------------------------------------------------------------------
versions
-------------------------------------------------------------------------
Request versions of binaries available for downloading.

* **--server** - address of the update server.
* **--version** - can be used to check the availability of a specific version of the binary.
