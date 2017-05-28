################################################################################
Description of the api requests
################################################################################

The API allows creating private keys (wallets) on request to receive funds and then send them to other wallets. All the private keys created are stored in an encrypted form.

********************************************************************************
Startup parameters
********************************************************************************

In order to work with this API, you need to specify the following parameters when starting EGAAS.

**-boltDir** - a directory where the file containing private keys will be created and stored. NoSQL database BoltDB is used to store the keys. The file is named *exchangeapi.db*. If the parameter is not specified, the file will be created in the current directory.

.. code:: 
      
      -boltDir=/home/temp
      
**-boltPsw**  - password for encrypting private keys when writing to the database. The API will not work if the password was not specified at startup. The password specified at first startup should be specified at subsequent startups. Don’t forget the password because it is not saved anywhere. If you specify an incorrect password or you enter the word "console" as a password, then you will be prompted to enter the password in the console after running EGAAS. Also, if the password has already been set but not specified in the command line parameter, it will be requested for in the console.

.. code:: 

      -boltPsw=mypass344
      
**-apiToken**  - this parameter specifies the token that will need to be passed when making a request to the API. The token specified will be saved, and can be omitted in subsequent startups. If this parameter has not been specified, then you will be able to call API commands without specifying the token parameter. If you'll need to change the token, you should start EGAAS with a new value in this parameter.

.. code:: 

      -apiToken=qweuytwuy347834
      
********************************************************************************
API requests
********************************************************************************

Responses to API requests are in JSON format and all of them have an error field. If this field is empty, then the request has been executed without errors. Otherwise, the field contains the text of the error that has occurred.

/exchangeapi/newkey
==============================
The command generates a private key, records it to a key file, and returns the public key and wallet addresses. For example:


*/exchange/newkey?token=qweuytwuy347834*

Response example:

.. code:: 

   {"error":"", "public":"b7880fa40779d673e7ea026377d7182744869c081b1d3a1d613fe661333ec67a74d985077555d80bc4aa65f5994f238def72881d6c2b6c60ffcc2ec7f050141d", "address":"0773-5161-7272-4133-0241", "wallet_id":7735161727241330241}

/exchange/send?sender=...&recipient=...&amount=...
==============================
The command sends money from the wallet (**sender**) from the database to the specified wallet (**recipient**). The wallets can be specified in any format  *XXXX-....-XXXX, int64, uint64*. It should be noted that the command only sends the transaction, but does not wait for confirmation that the money has been transferred. The value of the amount  (**amount**) should be indicated in  *qEGS*.

For example,

*/exchange/send?sender=1693-7869-8202-2463-0602&recipient=-3521799150320731671&amount=999000000000000*

Response example:

.. code:: 

     {"error":""}


/exchangeapi/balance?wallet=....
==============================
The command returns the balance of any wallet.

For example,

*/exchangeapi/balance?wallet=0773-5161-7272-4133-0241*

Response example:

.. code:: 

     {"error":"","amount":"99992318000000000000","egs":"99.992318"}

/exchangeapi/history?wallet=...&count=...
==============================
The command returns the last history of flow of funds in the specified wallet. count is an optional parameter and determines the number of records to be returned (1 more can be returned). By default, 50 entries will be returned, and the maximum number is 200.

For example,

*/exchangeapi/history?wallet=1693-7869-8202-2463-0602&count=10*

Response example:

.. code:: 

    {"error":"","history":[{"block_id":"118855","dif":"-0.001","amount":"99992318000000000000","egs":"99.992318","time":"03.05.2017 10:48:14"},{"block_id":"118855","dif":"-0.001999","amount":"99993318000000000000","egs":"99.993318","time":"03.05.2017 10:48:14"},{"block_id":"112283","dif":"-0.001","amount":"99995317000000000000","egs":"99.995317","time":"02.05.2017 18:28:24"}]}


