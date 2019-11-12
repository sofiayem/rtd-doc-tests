.. _building_requester:

===================================
Tutorial: Building an API Requester
===================================

.. TODO: Put the relevant links to API specification everywhere necessary.

This tutorial walks you through the process of creating a requester to Insolar API. You will learn how to **form and sign requests** that create a member capable of transferring funds to other members.

.. _what_you_will_build:

What You Will Build
-------------------

You will build a program (requester) that creates a member in the Insolar network and, as a member, transfers funds from its account.

The requester forms and sends the following requests to the Insolar’s JSON RPC 2.0 API endpoint and receives the corresponding responses:

#. Gets a **seed** that enables you to call a contract method.

#. Forms, signs, and sends a **member creation** request with the seed and receives your member’s reference in response.

#. Forms, signs, and sends a **transfer** request with another seed that sends an amount of Insolar coins (XNS):

   * from your member’s account given your reference received in the previous response;
   * to another’s account given a reference to the recipient member.

.. _what_you_will_need:

What You Will Need
------------------

* About an hour.
* Your favorite IDE for either:

  * Golang,
  * or Java.

* Insolar API specification as a reference.
* :ref:`Local Insolar deployment <setting_up_devnet>` or testing environment provided by Insolar.

.. _how_to_complete:

How to Complete This Tutorial
-----------------------------

* To start from scratch, go through the :ref:`step-by-instructions <build_requester>` listed below and:

  #. Click on the tabs that correspond to your programming language of choice (currently, Golang or Java) to see relevant code examples.
  #. Pay attention to comments in code examples.

* To skip the basics, read (and copy-paste) the working :ref:`requester code <requester_example>` provided at the end.

.. _build_requester:

Building the Requester
----------------------

To build the requester, go through the following steps:

#. **Prepare**: install the necessary tools, import the necessary packages, and initialize an HTTP client.

#. **Declare** request **structures** (Golang) or **classes** (Java) in accordance with the Insolar’s API specification.

#. **Create a seed getter** function. The getter will be reused to put a new seed into every signed request.

#. **Create a sender** function that signs and sends requests given a private key.

#. **Generate a key pair**, export the private key into a file, and store it in some secure place.

#. **Form a member creation request** and call the sender function to send it.

#. **Form a transfer request** and, again, call the sender function.

#. **Test** the requester against the testing environment.

All the above steps are detailed in sections below.

.. _prepare:

Step 1: Prepare
~~~~~~~~~~~~~~~

To build the requester, install, import, and set up the following:

#. Set up your development environment if you do not have one. Install one of the following:

   * `Go programming tools <https://golang.org/doc/install>`_;
   * `Java programming tools <https://java.com/en/download/help/download_options.xml>`_.

   .. note:: In Java, building this project via Maven requires a ``pom.xml`` file. A sample of this file is provided in the :ref:`testing section <test_requester>`.

#. With the Golang or Java programming tools, you do not need to “reinvent the wheel”: create a ``Main.go`` or ``Main.java`` file and, inside, import the packages your requester will use. For example:

   .. content-tabs::

      .. tab-container:: Golang
         :title: Golang: Main.go

         .. code-block:: Go
            :linenos:

            package main

            import (
               // You will need:
               // - Some basic Golang functionality.
               "os"
               "bytes"
               "io/ioutil"
               "fmt"
               "log"
               "strconv"
               // - HTTP client.
               "net/http"
               // - Big numbers to store signatures.
               "math/big"
               // - Basic cryptography.
               "crypto/x509"
               "crypto/elliptic"
               "crypto/ecdsa"
               "crypto/rand"
               "crypto/sha256"
               // - Basic encoding capabilities.
               "encoding/pem"
               "encoding/json"
               "encoding/base64"
               "encoding/asn1"
            )

      .. tab-container:: Java
         :title: Java: Main.java

         .. code-block:: Java
            :linenos:

            package requester;

            // You will need:
            // - Some basic Java functionality.
            import java.util.*;
            import java.nio.charset.StandardCharsets;
            import java.io.*;
            // - HTTP client.
            import java.net.URL;
            import java.net.http.HttpClient;
            import java.net.http.HttpRequest;
            import java.net.http.HttpResponse;
            // - Big numbers to store signatures.
            import java.math.BigInteger;
            // - Basic cryptography.
            import java.security.*;
            import java.security.spec.ECGenParameterSpec;
            import org.bouncycastle.asn1.*;
            import org.bouncycastle.openssl.jcajce.JcaPEMWriter;
            // - Basic encoding capabilities.
            import com.google.gson.Gson;
            import com.google.gson.annotations.SerializedName;
            import org.json.JSONObject;

#. To prepare the requester, do the following:

   #. Depending on the programming language:

      .. _golang_sig:

      * (**Golang**) Insolar supports ECDSA-signed requests. Since an ECDSA signature in Golang consists of two big integers, declare a single structure to contain it.
      * (**Java**) Since the program has to contain a main class, declare it to wrap all the required functionality.

      .. _set_url:

   #. Set the API endpoint URL for the testing environment, either the public one provided by Insolar or :ref:`locally deployed <setting_up_devnet>`.
   #. Create and initialize an HTTP client for connection re-use.
   #. Create a variable for the JSON RPC 2.0 request identifier. The identifier is to be incremented for every request and each corresponding response will contain it.

   For example:

   .. content-tabs::

      .. tab-container:: Golang
         :title: Golang: Main.go

         .. code-block:: Go
            :linenos:

            // Declare a structure to contain the ECDSA signature:
            type ecdsaSignature struct {
               R, S *big.Int
            }

            // Set the endpoint URL for the testing environment:
            const (
               url = "http://127.0.0.1:19101/api/rpc"
            )

            // Create and initialize an HTTP client for connection re-use:
            var client *http.Client

            func init() {
               client = &http.Client{}
            }

            // Create a variable for the JSON RPC 2.0 request identifier:
            var id int = 1
            // The identifier is to be incremented for every request and each corresponding response will contain it.

      .. tab-container:: Java
         :title: Java: Main.java

         .. code-block:: Java
            :linenos:

            // Declare a main class to wrap all the required functionality:
            public class Main {

                // Set the endpoint URL for the testing environment:
                private static final String API_URL = "http://127.0.0.1:19101/api/rpc";

                // Create and initialize an HTTP client for connection re-use:
                private static final HttpClient client = HttpClient.newBuilder().build();

                // Create a variable for the JSON RPC 2.0 request identifier:
                static Integer id = 1;
                // The identifier is to be incremented for every request and each corresponding response will contain it.

                // The Main class is to be continued...

With that, everything your requester needs is set up.

.. _declare_structs_or_classes:

Step 2: Declare Request Structures or Classes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, declare request structures (Golang) or classes (Java) in accordance with the Insolar’s API specification.

To transfer funds, you need structures or classes for:

#. Information request: ``node.getSeed``.
#. Contract requests: ``member.create`` and ``member.transfer``.

Both information and contract requests have the same base structure in accordance with the `JSON RPC 2.0 specification <https://www.jsonrpc.org/specification>`_.
Therefore, define the base structure once and expand it for all requests with their specific fields.

For example:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. code-block:: Go
         :linenos:

         // Continue in the Main.go file...

         // Declare a nested structure to form requests to Insolar API in accordance with the specification.
         // The Platform uses the basic JSON RPC 2.0 request structure:
         type requestBody struct {
            JSONRPC        string         `json:"jsonrpc"`
            ID             int            `json:"id"`
            Method         string         `json:"method"`
            // Params is a structure that depends on a particular method:
            Params         interface{}    `json:"params"`
         }

         // The Platform defines params of the signed request as follows:
         type params struct {
            Seed            string       `json:"seed"`
            CallSite        string       `json:"callSite"`
            // CallParams is a structure that depends on a particular method.
            CallParams      interface{}  `json:"callParams"`
            PublicKey       string       `json:"publicKey"`
         }

         type paramsWithReference struct {
            params
            Reference       string  `json:"reference"`
         }

         // The member.create request has no parameters, so it's an empty structure:
         type memberCreateCallParams struct {}

         // The transfer request sends an amount of funds to member identified by a reference:
         type transferCallParams struct {
            Amount            string    `json:"amount"`
            ToMemberReference string    `json:"toMemberReference"`
         }

   .. tab-container:: Java
      :title: Java: Main.java

      .. tip:: In Java, create the corresponding setters to initialize class instances later.

      .. code-block:: Java
         :linenos:

         // Continue in the Main class...

         // Declare a class to build a request:
         public static class Schema {

           // Declare a class to form requests to Insolar API in accordance with the specification.
           // The Platform uses the basic JSON RPC 2.0 request structure:
           public static class requestBody {
               @SerializedName("jsonrpc")
               private String jsonrpc;
               @SerializedName("id")
               private Integer id;
               @SerializedName("method")
               private String method;
               @SerializedName("params")
               private Params params;

               // Create setters for the variables:
               public requestBody() {
                   // Set the JSON RPC protocol version:
                   jsonrpc = "2.0";
                   id = 1;
                   method = null;
                   params = null;
               }
               public requestBody withID(Integer id) {
                   this.id = id;
                   return this;
               }

               public requestBody withMethod(String method) {
                   this.method = method;
                   return this;
               }

               // Params is a class which structure depends on a particular method:
               public requestBody withParams(Params params) {
                   this.params = params;
                   return this;
               }

               // Create a converter function to JSON:
               public String toJson() {
                   return new Gson().toJson(this);
               }
           }

           // The Platform defines params of the signed request as follows:
           public static class Params {

               @SerializedName("seed")
               private String seed;
               @SerializedName("callSite")
               private String callSite;
               // callParams is a structure that depends on a particular method.
               @SerializedName("callParams")
               private Object callParams;
               @SerializedName("reference")
               private String reference;
               @SerializedName("publicKey")
               private String publicKey;

               // Create the corresponding setters:
               public void setSeed(String seed) {
                   this.seed = seed;
               }

               public void setCallSite(String callSite) {
                   this.callSite = callSite;
               }

               public void setCallParams(Object callParams) {
                   this.callParams = callParams;
               }

               public void setReference(String reference) {
                   this.reference = reference;
               }

               public void setPublicKey(String publicKey) {
                   this.publicKey = publicKey;
               }
           }
           // The transfer request sends an amount of funds to member identified by a reference:
           public static class TransferCallParams {
               private String amount;
               private String toMemberReference;

               // Create the corresponding setter:
               public TransferCallParams(String amount, String toMemberReference) {
                   this.amount = amount;
                   this.toMemberReference = toMemberReference;
               }
           }
         }

Now that the requester knows which information and contract requests it is supposed to send, create the following functions:

#. Seed getter for the information request.
#. Sender for contract requests.

.. _create_seed_getter:

Step 3: Create a Seed Getter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each signed request to Insolar API has to contain a seed in its body. Seed is a unique piece of information generated by a node that:

* has a short lifespan;
* expires upon first use;
* protects from request fakes, i.e., duplicate requests sent by a malicious party sniffing the traffic. This ensures that a *single funds transfer request* can only be sent once by the owner of the funds.

.. tip:: Due to these qualities, a new seed is required to form each signed contract request.

.. caution:: Since the seed is generated by a node, each subsequent contract request containing the seed must be sent to the node in question. Otherwise, a node will reject the seed generated by a different one. To ensure that the contract request is routed to the correct node, retrieve a cookie from a response to the seed request. Later, the cookie is to be passed to the contract request sender function.

To be able to send signed requests, create a seed getter function to re-use upon forming each such request.

The seed getter:

#. Forms a ``node.getSeed`` request body in JSON format.
#. Creates an *unsigned* HTTP request with the body and a Content-Type (``application/json``) HTTP header.
#. Sends the request and receives a response.
#. Retrieves the acquired seed and cookie from the response and returns them.

For example:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. code-block:: Go
         :linenos:

         // Continue in the Main.go file...

         // Create a function to get a new seed for each signed request:
         func getNewSeed() (string, *http.Cookie) {
            // Form a request body for getSeed:
            getSeedReq := requestBody{
               JSONRPC: "2.0",
               Method:  "node.getSeed",
               ID:      id,
            }
            // Increment the id for future requests:
            id++

            // Marshal the payload into JSON:
            jsonSeedReq, err := json.Marshal(getSeedReq)
            if err != nil {
               log.Fatalln(err)
            }

            // Create a new HTTP request and send it:
            seedReq, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonSeedReq))
            if err != nil {
               log.Fatalln(err)
            }
            seedReq.Header.Set("ContentType", "application/json")

            // Perform the request:
            seedResponse, err := client.Do(seedReq)
            if err != nil {
               log.Fatalln(err)
            }
            defer seedReq.Body.Close()

            // Retrieve the cookie to pass it to a subsequent request and route the request to the correct node:
            var cookie *http.Cookie
            cookies := seedResponse.Cookies()
            if(len(cookies) > 0) {
               cookie = cookies[0]
            }

            // Receive the response body:
            seedRespBody, err := ioutil.ReadAll(seedResponse.Body)
            if err != nil {
               log.Fatalln(err)
            }

            // Unmarshal the response:
            var newSeed map[string]interface{}
            err = json.Unmarshal(seedRespBody, &newSeed)
            if err != nil {
               log.Fatalln(err)
            }

            // (Optional) Print the request and its response:
            print := "POST to " + url +
               "\nPayload: " + string(jsonSeedReq) +
               "\nResponse status code: " +  strconv.Itoa(seedResponse.StatusCode) +
               "\nResponse: " + string(seedRespBody) + "\n"
            fmt.Println(print)

            // Retrieve and return the current seed and cookie:
            return newSeed["result"].(map[string]interface{})["seed"].(string), cookie
         }

   .. tab-container:: Java
      :title: Java: Main.java

      .. tip:: In Java, to return multiple values (seed and cookie), define a class with the appropriate structure and make the function return its instance.

      .. code-block:: Java
         :linenos:

         // Continue in the Main class...

         // Create a class for the seed getter's return values (seed and cookie):
         public static class Pair {
           private String seed;
           private String cookie;

           public Pair(String seed, String cookie) {
               this.seed = seed;
               this.cookie = cookie;
           }

           public String getSeed() {
               return seed;
           }

           public String getCookie() {
               return cookie;
           }
         }

         // Create a function to get a new seed for each signed request:
         private static Pair getNewSeed() throws Exception {
           // Form a request body for getSeed and format it into JSON:
           String seedRequest = new Schema.requestBody().withMethod("node.getSeed").withID(id).toJson();
           // Increment the id for future requests:
           id++;

           // Create a new HTTP request and send it:
           URL url = new URL(API_URL);
           HttpRequest request = HttpRequest.newBuilder()
                   .POST(HttpRequest.BodyPublishers.ofString(seedRequest))
                   .header("Content-Type", "application/json; utf-8")
                   .uri(url.toURI())
                   .build();
           HttpResponse<String> send = client.send(request, HttpResponse.BodyHandlers.ofString());

           assert send.statusCode() == 200;

           // Receive the response body:
           String response = send.body();

           // (Optional) Print the request and its response:
           String req = new StringBuilder("\n\nPOST to ").append(url)
                   .append("\n")
                   .append("Payload: ")
                   .append(seedRequest)
                   .append("\nResponse status code: ").append(send.statusCode())
                   .append("\nResponse: ").append(response)
                   .append("\n")
                   .toString();
           System.out.println(req);

           // Retrieve and return the current seed and cookie:
           String seed =  new JSONObject(response).getJSONObject("result").getString("seed");
           String cookie = send.headers().firstValue("Set-Cookie").orElse(null);
           return new Pair(seed, cookie);
         }


Now, every ``getNewSeed()`` call will return a living seed that can be put into the contract request body and a cookie to ensure that the subsequent contract request is routed to the correct node.

The next step is to create a sender function that signs and sends contract requests.

.. _create_sender:

Step 4: Create a Sender Function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The sender function:

#. Takes some request body (payload), ECDSA private key, and the cookie retrieved from previous seed request. The cookie ensures that the contract request is to be routed to the node that generated the seed.
#. Forms an HTTP request with the payload and relevant HTTP headers:

   #. *Content-Type* — ``application/json``.
   #. *Digest* that contains (1) a SHA-256 hash of the payload's bytes (2) represented as a Base64 string.
   #. *Signature* that contains (1) the ECDSA signature of the hash's bytes (2) in the ASN.1 DER format (3) represented as a Base64 string.

#. Sends the request.
#. Returns the response JSON object.

For example:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. tip:: In Golang, the ECDSA signature consists of two big integers. To convert the signature into the ASN.1 DER format, put it into the ``ecdsaSignature`` structure defined in :ref:`one of the preparation steps <golang_sig>`.

      .. code-block:: Go
         :linenos:

         // Continue in the Main.go file...

         // Create a function to send signed requests:
         func sendSignedRequest(payload requestBody, privateKey *ecdsa.PrivateKey, cookie *http.Cookie) map[string]interface{} {
            // Marshal the payload into JSON:
            jsonPayload, err := json.Marshal(payload)
            if err != nil {
               log.Fatalln(err)
            }

            // Take a SHA-256 hash of the payload's bytes:
            hash := sha256.Sum256(jsonPayload)

            // Sign the hash with the private key:
            r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:])
            if err != nil {
               log.Fatalln(err)
            }

            // Convert the signature into ASN.1 DER format:
            sig := ecdsaSignature{
               R: r,
               S: s,
            }
            signature, err := asn1.Marshal(sig)
            if err != nil {
               log.Fatalln(err)
            }

            // Convert both hash and signature into a Base64 string:
            hash64 := base64.StdEncoding.EncodeToString(hash[:])
            signature64 := base64.StdEncoding.EncodeToString(signature)

            // Create a new request and set its headers:
            request, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonPayload))
            if err != nil {
               log.Fatalln(err)
            }
            request.Header.Set("ContentType", "application/json")

            // Put the hash string into the HTTP Digest header:
            request.Header.Set("Digest", "SHA-256="+hash64)

            // Put the signature string into the HTTP Signature header:
            request.Header.Set("Signature", "keyId=\"public-key\", algorithm=\"ecdsa\", headers=\"digest\", signature="+signature64)

             // Set cookie to route the request to the node that generated the seed:
            request.AddCookie(cookie)

            // Send the signed request:
            response, err := client.Do(request)
            if err != nil {
               log.Fatalln(err)
            }
            defer response.Body.Close()

            // Receive the response body:
            responseBody, err := ioutil.ReadAll(response.Body)
            if err != nil {
               log.Fatalln(err)
            }

            // Unmarshal it into a JSON object:
            var JSONObject map[string]interface{}
            err = json.Unmarshal(responseBody, &JSONObject)
            if err != nil {
               log.Fatalln(err)
            }

            // (Optional) Print the request and its response:
            print := "POST to " + url +
               "\nPayload: " + string(jsonPayload) +
               "\nResponse status code: " + strconv.Itoa(response.StatusCode) +
               "\nResponse: " + string(responseBody) + "\n"
            fmt.Println(print)

            // Return the response:
            return JSONObject
         }

   .. tab-container:: Java
      :title: Java: Main.java

      .. code-block:: Java
         :linenos:

         // Continue in the Main class...

         // Create a function to send signed requests:
         private static JSONObject sendSignedRequest(String requestBody, PrivateKey privateKey, String cookie) throws Exception {

           // Take a SHA-256 hash of the payload's bytes:
           byte[] payload = requestBody.getBytes("UTF-8");
           MessageDigest detester = MessageDigest.getInstance("SHA-256");
           detester.update(payload);
           byte[] digest = detester.digest();

           // Sign the hash with the private key:
           Signature ecdsaSign = Signature.getInstance("SHA256withECDSA", "BC");
           ecdsaSign.initSign(privateKey);
           ecdsaSign.update(payload);
           byte[] signature = ecdsaSign.sign();

           // Convert the signature into ASN.1 DER format:
           ASN1InputStream asn1 = new ASN1InputStream(signature);
           DLSequence dlSequence = (DLSequence) asn1.readObject();
           BigInteger r = ((ASN1Integer) dlSequence.getObjectAt(0)).getPositiveValue();
           BigInteger s = ((ASN1Integer) dlSequence.getObjectAt(1)).getPositiveValue();
           ByteArrayOutputStream bos = new ByteArrayOutputStream();
           DERSequenceGenerator seq = new DERSequenceGenerator(bos);
           seq.addObject(new ASN1Integer(r));
           seq.addObject(new ASN1Integer(s));
           seq.close();
           byte[] derSignature = bos.toByteArray();

           // Convert both hash and signature into a Base64 string:
           String digest64 = Base64.getEncoder().encodeToString(digest);
           String signature64 = Base64.getEncoder().encodeToString(derSignature);

           // Put the hash string into the HTTP Digest header:
           String digestHeader = "SHA-256=" + digest64;
           // Put the signature string into the HTTP Signature header:
           String signatureHeader = "keyId=\"member-pub-key\", algorithm=\"ecdsa\", headers=\"digest\", signature=" + signature64;

           // Create a new request, pass the cookie to route the request to the correct node, and send it:
           URL url = new URL(API_URL);
           HttpRequest request = HttpRequest.newBuilder()
                   .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                   .header("Content-Type", "application/json; utf-8")
                   .header("Digest", digestHeader)
                   .header("Signature", signatureHeader)
                   .header("Cookie", cookie)
                   .uri(url.toURI())
                   .build();
           HttpResponse<String> send = client.send(request, HttpResponse.BodyHandlers.ofString());

           assert send.statusCode() == 200;

           // Receive the response:
           String response = send.body();

           // (Optional) Print the request and its response:
           String req = new StringBuilder("\n\nPOST to ").append(url)
                   .append("\n")
                   .append("Payload: ")
                   .append(requestBody)
                   .append("\nResponse status code = ").append(send.statusCode())
                   .append("\nResponse: ").append(response)
                   .append("\n")
                   .toString();
           System.out.println(req);

           // Return the response:
           return new JSONObject(response);
         }

Now, every ``sendSignedRequest(payload, privateKey, cookie)`` call will return the result of a contract method.

With the seed getter and sender functions, you can get the seed and send signed contract requests. The next step is to generate a key pair.

.. _generate_key_pair:

Step 5: Generate a Key Pair
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The body of each request that calls a contract method must be hashed by a ``SHA256`` algorithm. Each hash must be signed by a private key generated by a ``P256`` (Golang) or a corresponding ``secp256k1`` (Java) elliptic curve.

To be able to sign requests, do the following:

#. Generate a key pair using the said curve and convert it into PEM format.

   .. warning:: You will not be able to access your member object without the private key and, as such, transfer funds.

#. Export the private key into a file.
#. Save the file into some secure place.

For example:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. tip:: In Golang, to encode the key into the PEM format, first, convert it into ASN.1 DER using the ``x509`` library.

      .. code-block:: Go
         :linenos:

         // Continue in the Main.go file...

         // Create the main function:
         func main() {
            // Generate a key pair:
            privateKey := new(ecdsa.PrivateKey)
            privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
            var publicKey ecdsa.PublicKey
            publicKey = privateKey.PublicKey

            // Convert both private and public keys into PEM format:
            x509PublicKey, err := x509.MarshalPKIXPublicKey(&publicKey)
            if err != nil {
                log.Fatalln(err)
            }
            pemPublicKey := pem.EncodeToMemory(&pem.Block{Type: "PUBLIC KEY", Bytes: x509PublicKey})

            x509PrivateKey, err := x509.MarshalECPrivateKey(privateKey)
            if err != nil {
                log.Fatalln(err)
            }
            pemPrivateKey := pem.EncodeToMemory(&pem.Block{Type: "PRIVATE KEY", Bytes: x509PrivateKey})

            // The private key is required to sign requests.
            // Make sure to put into a file to save it in some secure place later:
            file, err := os.Create("private.pem")
            if err != nil {
                fmt.Println(err)
                return
            }
            file.WriteString(string(pemPrivateKey))
            file.Close()

            // The main function is to be continued...
          }

   .. tab-container:: Java
      :title: Java: Main.java

      .. code-block:: Java
         :linenos:

         // Continue in the Main class...

         // Create the main function:
         public static void main(String[] args) throws Exception {
             // Generate a key pair:
             Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());
             SecureRandom secureRandom = new SecureRandom();
             ECGenParameterSpec spec = new ECGenParameterSpec("secp256k1");
             KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("ECDSA");
             keyPairGenerator.initialize(spec, secureRandom);
             KeyPair keyPair = keyPairGenerator.generateKeyPair();

             // Convert the public key into PEM format:
             ByteArrayOutputStream baos = new ByteArrayOutputStream();
             JcaPEMWriter jcaPEMWriter = new JcaPEMWriter(new OutputStreamWriter(baos, StandardCharsets.UTF_8));
             jcaPEMWriter.writeObject(keyPair.getPublic());
             jcaPEMWriter.flush();
             jcaPEMWriter.close();
             String publicKey = new String(baos.toByteArray());

             // The private key is required to sign requests.
             // Convert it into PEM format and make sure to put into a file to save it in some secure place later:
             StringWriter stringWriter = new StringWriter();
             JcaPEMWriter pemWriter = new JcaPEMWriter(stringWriter);
             try (PrintStream out = new PrintStream(new FileOutputStream("private.pem"))) {
                 pemWriter.writeObject(keyPair);
                 pemWriter.close();
                 String pem = stringWriter.toString();
                 out.print(pem);
             }

             // The main function is to be continued...
          }

Now that the key pair is generated and saved, you can form contract requests.

.. _form_member_create:

Step 6: Form and Send a Member Creation Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The member creation request is a signed request to a contract method that does the following in the blockchain:

* Creates a new member and corresponding account objects.
* Returns the new member reference — address in the Insolar network.
* Binds a given public key to the member. Insolar uses this public key to identify a member and check the signature generated by the paired private key.

To create a member:

#. Call the ``getNewSeed()`` function and store the new seed into a variable.
#. Form the ``member.create`` request payload with the seed and the public key generated in the :ref:`previous step <generate_key_pair>`.
#. Call the ``sendSignedRequest()`` function and pass it the payload, private key, and cookie.
#. Put the returned member reference into a variable. The subsequent transfer request requires it.

For example:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. code-block:: Go
         :linenos:

         // Continue in the main() function...

         // Get a seed and cookie to form the request:
         seed, cookie := getNewSeed()
         // Form a request body for member.create:
         createMemberReq := requestBody{
            JSONRPC: "2.0",
            Method:  "contract.call",
            ID:      id,
            Params:params {
               Seed: seed,
               CallSite: "member.create",
               CallParams:memberCreateCallParams {},
               PublicKey: string(pemPublicKey),},
         }
         // Increment the JSON RPC 2.0 request identifier for future requests:
         id++

         // Send the signed member.create request and pass the cookie:
         newMember := sendSignedRequest(createMemberReq, privateKey, cookie)
         // Put the reference to your new member into a variable to send transfer requests:
         memberReference := newMember["result"].(map[string]interface{})["callResult"].(map[string]interface{})["reference"].(string)
         fmt.Println("Member reference is " + memberReference)

         // The main function is to be continued...

   .. tab-container:: Java
      :title: Java: Main.java

      .. code-block:: Java
         :linenos:

         // Continue in the main() function...

         // Get a seed and cookie to form a request:
         Pair seedAndCookie = getNewSeed();
         String seed = seedAndCookie.getSeed();
         String cookie = seedAndCookie.getCookie();
         // Form a request body for member.create:
         Schema.Params memberParams = new Schema.Params();
         memberParams.setSeed(seed);
         memberParams.setCallSite("member.create");
         memberParams.setPublicKey(publicKey);

         // Form a JSON payload:
         String createMemberReq = new Schema.requestBody().withMethod("contract.call").withParams(memberParams).withID(id).toJson();

         // Increment the JSON RPC 2.0 request identifier for future requests:
         id++;

         // Send the signed member.create request and pass the cookie:
         JSONObject newMember = sendSignedRequest(createMemberReq, keyPair.getPrivate(), cookie);
         assert newMember.isNull("error");

         // Put the reference to your new member into a variable to send subsequent transfer requests:
         String memberReference = newMember.getJSONObject("result").getJSONObject("callResult").getString("reference");
         System.out.println("Member reference is " + memberReference);

         // The main function is to be continued...

Now that you have your member reference, you can transfer funds to other members.

.. _form_transfer:

Step 7: Form and Send a Transfer Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The transfer request is a signed request to a contract method that transfers some amount of funds to another member.

To transfer funds:

#. Acquire the recipient reference — the reference to a member to whom you want to transfer the funds.
#. Call the ``getNewSeed()`` function and store the new seed into a variable.
#. Form a ``member.transfer`` request payload with:

   * a new seed and cookie,
   * an amount of funds to transfer,
   * the recipient reference,
   * your reference (for identification),
   * and your public key (to check the signature).

#. Call the ``sendSignedRequest()`` function and pass it the payload, paired private key, and cookie.

The transfer request will return the factual fee value in its response.

For example:

.. attention:: In the highlighted line, replace the ``<recipient_member_reference>`` placeholder value with the reference to the recipient member.

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. code-block:: Go
         :linenos:
         :emphasize-lines: 15

         // Continue in the main() function...

         // Get a new seed and cookie to form a transfer request:
         seed, cookie = getNewSeed()
         // Form a request body for transfer:
         transferReq := requestBody{
            JSONRPC: "2.0",
            Method:  "contract.call",
            ID:      id,
            Params:paramsWithReference{ params:params{
               Seed: seed,
               CallSite: "member.transfer",
               CallParams:transferCallParams {
                  Amount: "100",
                  ToMemberReference: "<recipient_member_reference>",
               },
               PublicKey: string(pemPublicKey),
            },
               Reference: string(memberReference),
            },
         }
         // Increment the id for future requests:
         id++

         // Send the signed transfer request and pass the cookie:
         newTransfer := sendSignedRequest(transferReq, privateKey, cookie)
         fee := newTransfer["result"].(map[string]interface{})["callResult"].(map[string]interface{})["fee"].(string)

         // (Optional) Print out the fee.
         fmt.Println("Fee is " + fee)

         // Remember to close the main function.
         }

   .. tab-container:: Java
      :title: Java: Main.java

      .. code-block:: Java
         :linenos:
         :emphasize-lines: 13

         // Continue in the main() function...

         // Get a new seed to form a transfer request:
         seedAndCookie = getNewSeed();
         seed = seedAndCookie.getSeed();
         cookie = seedAndCookie.getCookie();

         // Form a request body for transfer:
         Schema.Params transferParams = new Schema.Params();
         transferParams.setSeed(seed);
         transferParams.setCallSite("member.transfer");
         transferParams.setPublicKey(publicKey);
         transferParams.setCallParams(new Schema.TransferCallParams("100", "<recipient_member_reference>"));
         transferParams.setReference(memberReference);

         // Form a JSON payload:
         String transferReq = new Schema.requestBody().withMethod("contract.call").withParams(transferParams).withID(id).toJson();

         // Increment the id for future requests:
         id++;

         // Send the signed transfer request:
         JSONObject newTransfer = sendSignedRequest(transferReq, keyPair.getPrivate(), cookie);
         assert newTransfer.isNull("error");
         String fee = newTransfer.getJSONObject("result").getJSONObject("callResult").getString("fee");

         // (Optional) Print out the fee.
         System.out.println("Fee is " + fee);

         // Close the main() function.
         }
         // And remember to close the Main class.
         }

With that, the requester, as a member, can send funds to other members of the Insolar network.

.. _test_requester:

Step 8: Test the Requester
~~~~~~~~~~~~~~~~~~~~~~~~~~

To test the requester, do the following:

#. Make sure the :ref:`endpoint URL <set_url>` is set to that of the testing environment.
#. Run the requester:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang

      .. code-block:: console

         $ go run Main.go

   .. tab-container:: Java
      :title: Java

      3. In Java, create a ``pom.xml`` file to contain information about the project and configuration details used by Maven to build it:

         .. toggle-header::
           :header: ``pom.xml`` **Show/Hide**

           .. code-block:: xml
              :linenos:

              <?xml version="1.0" encoding="UTF-8"?>
              <project xmlns="http://maven.apache.org/POM/4.0.0"
                       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
                  <modelVersion>4.0.0</modelVersion>

                  <groupId>requester</groupId>
                  <artifactId>requester-java-example</artifactId>
                  <version>1.0-SNAPSHOT</version>

                  <properties>
                      <java.version>11</java.version>
                  </properties>

                  <dependencies>
                      <dependency>
                          <groupId>org.bouncycastle</groupId>
                          <artifactId>bcprov-jdk15on</artifactId>
                          <version>1.60</version>
                      </dependency>
                      <dependency>
                          <groupId>org.bouncycastle</groupId>
                          <artifactId>bcpkix-jdk15on</artifactId>
                          <version>1.51</version>
                      </dependency>
                      <dependency>
                          <groupId>org.json</groupId>
                          <artifactId>json</artifactId>
                          <version>20180813</version>
                      </dependency>
                      <dependency>
                          <groupId>com.google.code.gson</groupId>
                          <artifactId>gson</artifactId>
                          <version>2.8.5</version>
                      </dependency>
                  </dependencies>
                  
                  <build>
                      <plugins>
                          <plugin>
                              <groupId>org.apache.maven.plugins</groupId>
                              <artifactId>maven-compiler-plugin</artifactId>
                              <version>3.8.0</version>
                              <configuration>
                                  <release>11</release>
                              </configuration>
                          </plugin>

                          <plugin>
                              <groupId>org.apache.maven.plugins</groupId>
                              <artifactId>maven-assembly-plugin</artifactId>
                              <executions>
                                  <execution>
                                      <phase>package</phase>
                                      <goals>
                                          <goal>single</goal>
                                      </goals>
                                      <configuration>
                                          <archive>
                                              <manifest>
                                                  <mainClass>
                                                      requester.Main
                                                  </mainClass>
                                              </manifest>
                                          </archive>
                                          <descriptorRefs>
                                              <descriptorRef>jar-with-dependencies</descriptorRef>
                                          </descriptorRefs>
                                      </configuration>
                                  </execution>
                              </executions>
                          </plugin>

                      </plugins>
                  </build>
                  
              </project>

      4. Install and run:

         .. code-block:: console

            $ mvn install
            $ java -jar ./target/insolar-java-example-1.0-SNAPSHOT-jar-with-dependencies.jar

.. _Summary:

Summary
-------

Congratulations! You have just developed a requester capable of forming signed requests to interact with the Insolar API.

Build upon it:

#. Create structures for other requests in accordance with the Insolar API specification.
#. Export the getter and sender functions to use them in other packages.

.. _requester_example:

Complete Requester Code Examples
--------------------------------

Below are the complete requester code examples in both Golang and Java. Click the links to show or hide them.

.. attention:: To be able to send transfer requests, in the highlighted line, replace the ``<recipient_member_reference>`` placeholder value with the reference to the recipient member.

.. toggle-header::
   :header: Golang: ``Main.go`` file. **Show/Hide**

   .. code-block:: Go
      :linenos:
      :emphasize-lines: 289

      package main

      import (
        // You will need:
        // - Some basic Golang functionality.
        "os"
        "bytes"
        "io/ioutil"
        "fmt"
        "log"
        "strconv"
        // - HTTP client.
        "net/http"
        // - Big numbers to store signatures.
        "math/big"
        // - Basic cryptography.
        "crypto/x509"
        "crypto/elliptic"
        "crypto/ecdsa"
        "crypto/rand"
        "crypto/sha256"
        // - Basic encoding capabilities.
        "encoding/pem"
        "encoding/json"
        "encoding/base64"
        "encoding/asn1"
      )

      // Declare a structure to contain the ECDSA signature:
      type ecdsaSignature struct {
        R, S *big.Int
      }

      // Set the endpoint URL for the testing environment:
      const (
        url = "http://127.0.0.1:19101/api/rpc"
      )

      // Create and initialize an HTTP client for connection re-use:
      var client *http.Client

      func init() {
        client = &http.Client{}
      }

      // Create a variable for the JSON RPC 2.0 request identifier:
      var id int = 1
      // The identifier is to be incremented for every request and each corresponding response will contain it.

      // Declare a nested structure to form requests to Insolar API in accordance with the specification.
      // The Platform uses the basic JSON RPC 2.0 request structure:
      type requestBody struct {
        JSONRPC        string         `json:"jsonrpc"`
        ID             int            `json:"id"`
        Method         string         `json:"method"`
        // Params is a structure that depends on a particular method:
        Params         interface{}    `json:"params"`
      }

      // The Platform defines params of the signed request as follows:
      type params struct {
        Seed            string       `json:"seed"`
        CallSite        string       `json:"callSite"`
        // CallParams is a structure that depends on a particular method.
        CallParams      interface{}  `json:"callParams"`
        PublicKey       string       `json:"publicKey"`
      }

      type paramsWithReference struct {
        params
        Reference       string  `json:"reference"`
      }

      // The member.create request has no parameters, so it's an empty structure:
      type memberCreateCallParams struct {}

      // The transfer request sends an amount of funds to member identified by a reference:
      type transferCallParams struct {
        Amount            string    `json:"amount"`
        ToMemberReference string    `json:"toMemberReference"`
      }

      // Create a function to get a new seed for each signed request:
      func getNewSeed() (string, *http.Cookie) {
        // Form a request body for getSeed:
        getSeedReq := requestBody{
          JSONRPC: "2.0",
          Method:  "node.getSeed",
          ID:      id,
        }
        // Increment the id for future requests:
        id++

        // Marshal the payload into JSON:
        jsonSeedReq, err := json.Marshal(getSeedReq)
        if err != nil {
          log.Fatalln(err)
        }

        // Create a new HTTP request and send it:
        seedReq, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonSeedReq))
        if err != nil {
          log.Fatalln(err)
        }
        seedReq.Header.Set("ContentType", "application/json")

        // Perform the request:
        seedResponse, err := client.Do(seedReq)
        if err != nil {
          log.Fatalln(err)
        }
        defer seedReq.Body.Close()

        // Retrieve the cookie to use it in the subsequent request:
        var cookie *http.Cookie
        cookies := seedResponse.Cookies()
        if(len(cookies) > 0) {
          cookie = cookies[0]
        }

        // Receive the response body:
        seedRespBody, err := ioutil.ReadAll(seedResponse.Body)
        if err != nil {
          log.Fatalln(err)
        }

        // Unmarshal the response:
        var newSeed map[string]interface{}
        err = json.Unmarshal(seedRespBody, &newSeed)
        if err != nil {
          log.Fatalln(err)
        }

        // (Optional) Print the request and its response:
        print := "POST to " + url +
          "\nPayload: " + string(jsonSeedReq) +
          "\nResponse status code: " +  strconv.Itoa(seedResponse.StatusCode) +
          "\nResponse: " + string(seedRespBody) + "\n"
        fmt.Println(print)

        // Retrieve and return the current seed and cookie:
        return newSeed["result"].(map[string]interface{})["seed"].(string), cookie
      }

      // Create a function to send signed requests:
      func sendSignedRequest(payload requestBody, privateKey *ecdsa.PrivateKey, cookie *http.Cookie) map[string]interface{} {
        // Marshal the payload into JSON:
        jsonPayload, err := json.Marshal(payload)
        if err != nil {
          log.Fatalln(err)
        }

        // Take a SHA-256 hash of the payload's bytes:
        hash := sha256.Sum256(jsonPayload)

        // Sign the hash with the private key:
        r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:])
        if err != nil {
          log.Fatalln(err)
        }

        // Convert the signature into ASN.1 DER format:
        sig := ecdsaSignature{
          R: r,
          S: s,
        }
        signature, err := asn1.Marshal(sig)
        if err != nil {
          log.Fatalln(err)
        }

        // Convert both hash and signature into a Base64 string:
        hash64 := base64.StdEncoding.EncodeToString(hash[:])
        signature64 := base64.StdEncoding.EncodeToString(signature)

        // Create a new request and set its headers:
        request, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonPayload))
        if err != nil {
          log.Fatalln(err)
        }
        request.Header.Set("ContentType", "application/json")

        // Put the hash string into the HTTP Digest header:
        request.Header.Set("Digest", "SHA-256="+hash64)

        // Put the signature string into the HTTP Signature header:
        request.Header.Set("Signature", "keyId=\"public-key\", algorithm=\"ecdsa\", headers=\"digest\", signature="+signature64)

        // Set the cookie to route the contract request to the node that generated the seed:
        request.AddCookie(cookie)

        // Send the signed request:
        response, err := client.Do(request)
        if err != nil {
          log.Fatalln(err)
        }
        defer response.Body.Close()

        // Receive the response body:
        responseBody, err := ioutil.ReadAll(response.Body)
        if err != nil {
          log.Fatalln(err)
        }

        // Unmarshal it into a JSON object:
        var JSONObject map[string]interface{}
        err = json.Unmarshal(responseBody, &JSONObject)
        if err != nil {
          log.Fatalln(err)
        }

        // (Optional) Print the request and its response:
        print := "POST to " + url +
          "\nPayload: " + string(jsonPayload) +
          "\nResponse status code: " + strconv.Itoa(response.StatusCode) +
          "\nResponse: " + string(responseBody) + "\n"
        fmt.Println(print)

        // Return the response:
        return JSONObject
      }

      // Create the main function to form and send signed requests:
      func main() {
        // Generate a key pair:
        privateKey := new(ecdsa.PrivateKey)
        privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
        var publicKey ecdsa.PublicKey
        publicKey = privateKey.PublicKey

        // Convert both private and public keys into PEM format:
        x509PublicKey, err := x509.MarshalPKIXPublicKey(&publicKey)
        if err != nil {
          log.Fatalln(err)
        }
        pemPublicKey := pem.EncodeToMemory(&pem.Block{Type: "PUBLIC KEY", Bytes: x509PublicKey})

        x509PrivateKey, err := x509.MarshalECPrivateKey(privateKey)
        if err != nil {
          log.Fatalln(err)
        }
        pemPrivateKey := pem.EncodeToMemory(&pem.Block{Type: "PRIVATE KEY", Bytes: x509PrivateKey})

        // The private key is required to sign requests.
        // Make sure to put into a file to save it in some secure place later:
        file, err := os.Create("private.pem")
        if err != nil {
          fmt.Println(err)
          return
        }
        file.WriteString(string(pemPrivateKey))
        file.Close()

        // Get a seed and cookie to form the request:
        seed, cookie := getNewSeed()
        // Form a request body for member.create:
        createMemberReq := requestBody{
          JSONRPC: "2.0",
          Method:  "contract.call",
          ID:      id,
          Params:params {
            Seed: seed,
            CallSite: "member.create",
            CallParams:memberCreateCallParams {},
            PublicKey: string(pemPublicKey),},
        }
        // Increment the JSON RPC 2.0 request identifier for future requests:
        id++

        // Send the signed member.create request and pass the cookie:
        newMember := sendSignedRequest(createMemberReq, privateKey, cookie)

        // Put the reference to your new member into a variable to send transfer requests:
        memberReference := newMember["result"].(map[string]interface{})["callResult"].(map[string]interface{})["reference"].(string)
        fmt.Println("Member reference is " + memberReference)

        // Get a new seed and cookie to form a transfer request:
        seed, cookie = getNewSeed()
        // Form a request body for transfer:
        transferReq := requestBody{
          JSONRPC: "2.0",
          Method:  "contract.call",
          ID:      id,
          Params:paramsWithReference{ params:params{
            Seed: seed,
            CallSite: "member.transfer",
            CallParams:transferCallParams {
              Amount: "100",
              ToMemberReference: "<recipient_member_reference>",
              },
            PublicKey: string(pemPublicKey),
            },
            Reference: string(memberReference),
          },
        }
        // Increment the id for future requests:
        id++

        // Send the signed transfer request and pass the cookie:
        newTransfer := sendSignedRequest(transferReq, privateKey, cookie)
        fee := newTransfer["result"].(map[string]interface{})["callResult"].(map[string]interface{})["fee"].(string)

        // (Optional) Print out the fee.
        fmt.Println("Fee is " + fee)
      }

|

.. toggle-header::
   :header: Java: ``Main.java`` file. **Show/Hide**

   .. code-block:: Java
      :linenos:
      :emphasize-lines: 321

      package requester;

      // You will need:
      // - Some basic Java functionality.
      import java.util.*;
      import java.nio.charset.StandardCharsets;
      import java.io.*;
      // - HTTP client.
      import java.net.URL;
      import java.net.http.HttpClient;
      import java.net.http.HttpRequest;
      import java.net.http.HttpResponse;
      // - Big numbers to store signatures.
      import java.math.BigInteger;
      // - Basic cryptography.
      import java.security.*;
      import java.security.spec.ECGenParameterSpec;

      import org.bouncycastle.asn1.*;
      import org.bouncycastle.openssl.jcajce.JcaPEMWriter;
      // - Basic encoding capabilities.
      import com.google.gson.Gson;
      import com.google.gson.annotations.SerializedName;
      import org.json.JSONObject;

      // Declare a main class to wrap all the required functionality:
      public class Main {

          // Set the endpoint URL for the testing environment:
          private static final String API_URL = "http://127.0.0.1:19101/api/rpc";

          // Create and initialize an HTTP client for connection re-use:
          private static final HttpClient client = HttpClient.newBuilder().build();

          // Create a variable for the JSON RPC 2.0 request identifier:
          static Integer id = 1;
          // The identifier is to be incremented for every request and each corresponding response will contain it.

          // Declare a class to build a request:
          public static class Schema {

              // Declare a class to form requests to Insolar API in accordance with the specification.
              // The Platform uses the basic JSON RPC 2.0 request structure:
              public static class requestBody {
                  @SerializedName("jsonrpc")
                  private String jsonrpc;
                  @SerializedName("id")
                  private Integer id;
                  @SerializedName("method")
                  private String method;
                  @SerializedName("params")
                  private Params params;

                  // Create setters for the variables:
                  public requestBody() {
                      // Set the JSON RPC protocol version:
                      jsonrpc = "2.0";
                      id = 1;
                      method = null;
                      params = null;
                  }
                  public requestBody withID(Integer id) {
                      this.id = id;
                      return this;
                  }

                  public requestBody withMethod(String method) {
                      this.method = method;
                      return this;
                  }

                  // Params is a class which structure depends on a particular method:
                  public requestBody withParams(Params params) {
                      this.params = params;
                      return this;
                  }

                  // Create a converter function to JSON:
                  public String toJson() {
                      return new Gson().toJson(this);
                  }
              }

              // The Platform defines params of the signed request as follows:
              public static class Params {

                  @SerializedName("seed")
                  private String seed;
                  @SerializedName("callSite")
                  private String callSite;
                  // callParams is a structure that depends on a particular method.
                  @SerializedName("callParams")
                  private Object callParams;
                  @SerializedName("reference")
                  private String reference;
                  @SerializedName("publicKey")
                  private String publicKey;

                  // Create the corresponding setters:
                  public void setSeed(String seed) {
                      this.seed = seed;
                  }

                  public void setCallSite(String callSite) {
                      this.callSite = callSite;
                  }

                  public void setCallParams(Object callParams) {
                      this.callParams = callParams;
                  }

                  public void setReference(String reference) {
                      this.reference = reference;
                  }

                  public void setPublicKey(String publicKey) {
                      this.publicKey = publicKey;
                  }
              }
              // The transfer request sends an amount of funds to member identified by a reference:
              public static class TransferCallParams {
                  private String amount;
                  private String toMemberReference;

                  // Create the corresponding setter:
                  public TransferCallParams(String amount, String toMemberReference) {
                      this.amount = amount;
                      this.toMemberReference = toMemberReference;
                  }
              }
          }

          // Create a class for the seed getter's return values (seed and cookie):
          public static class Pair {
              private String seed;
              private String cookie;

              public Pair(String seed, String cookie) {
                  this.seed = seed;
                  this.cookie = cookie;
              }

              public String getSeed() {
                  return seed;
              }

              public String getCookie() {
                  return cookie;
              }
          }

          // Create a function to get a new seed for each signed request:
          private static Pair getNewSeed() throws Exception {
              // Form a request body for getSeed and format it into JSON:
              String seedRequest = new Schema.requestBody().withMethod("node.getSeed").withID(id).toJson();
              // Increment the id for future requests:
              id++;

              // Create a new HTTP request and send it:
              URL url = new URL(API_URL);
              HttpRequest request = HttpRequest.newBuilder()
                      .POST(HttpRequest.BodyPublishers.ofString(seedRequest))
                      .header("Content-Type", "application/json; utf-8")
                      .uri(url.toURI())
                      .build();
              HttpResponse<String> send = client.send(request, HttpResponse.BodyHandlers.ofString());

              assert send.statusCode() == 200;

              // Receive the response body:
              String response = send.body();

              // (Optional) Print the request and its response:
              String req = new StringBuilder("\n\nPOST to ").append(url)
                      .append("\n")
                      .append("Payload: ")
                      .append(seedRequest)
                      .append("\nResponse status code: ").append(send.statusCode())
                      .append("\nResponse: ").append(response)
                      .append("\n")
                      .toString();
              System.out.println(req);

              // Retrieve and return the current seed and cookie:
              String seed =  new JSONObject(response).getJSONObject("result").getString("seed");
              String cookie = send.headers().firstValue("Set-Cookie").orElse(null);
              return new Pair(seed, cookie);
          }

          // Create a function to send signed requests:
          private static JSONObject sendSignedRequest(String requestBody, PrivateKey privateKey, String cookie) throws Exception {

              // Take a SHA-256 hash of the payload's bytes:
              byte[] payload = requestBody.getBytes("UTF-8");
              MessageDigest detester = MessageDigest.getInstance("SHA-256");
              detester.update(payload);
              byte[] digest = detester.digest();

              // Sign the hash with the private key:
              Signature ecdsaSign = Signature.getInstance("SHA256withECDSA", "BC");
              ecdsaSign.initSign(privateKey);
              ecdsaSign.update(payload);
              byte[] signature = ecdsaSign.sign();

              // Convert the signature into ASN.1 DER format:
              ASN1InputStream asn1 = new ASN1InputStream(signature);
              DLSequence dlSequence = (DLSequence) asn1.readObject();
              BigInteger r = ((ASN1Integer) dlSequence.getObjectAt(0)).getPositiveValue();
              BigInteger s = ((ASN1Integer) dlSequence.getObjectAt(1)).getPositiveValue();
              ByteArrayOutputStream bos = new ByteArrayOutputStream();
              DERSequenceGenerator seq = new DERSequenceGenerator(bos);
              seq.addObject(new ASN1Integer(r));
              seq.addObject(new ASN1Integer(s));
              seq.close();
              byte[] derSignature = bos.toByteArray();

              // Convert both hash and signature into a Base64 string:
              String digest64 = Base64.getEncoder().encodeToString(digest);
              String signature64 = Base64.getEncoder().encodeToString(derSignature);

              // Put the hash string into the HTTP Digest header:
              String digestHeader = "SHA-256=" + digest64;
              // Put the signature string into the HTTP Signature header:
              String signatureHeader = "keyId=\"member-pub-key\", algorithm=\"ecdsa\", headers=\"digest\", signature=" + signature64;

              // Create a new request, pass the cookie to route it to the node that generated the seed, and send it:
              URL url = new URL(API_URL);
              HttpRequest request = HttpRequest.newBuilder()
                      .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                      .header("Content-Type", "application/json; utf-8")
                      .header("Digest", digestHeader)
                      .header("Signature", signatureHeader)
                      .header("Cookie", cookie)
                      .uri(url.toURI())
                      .build();
              HttpResponse<String> send = client.send(request, HttpResponse.BodyHandlers.ofString());

              assert send.statusCode() == 200;

              // Receive the response:
              String response = send.body();

              // (Optional) Print the request and its response:
              String req = new StringBuilder("\n\nPOST to ").append(url)
                      .append("\n")
                      .append("Payload: ")
                      .append(requestBody)
                      .append("\nResponse status code = ").append(send.statusCode())
                      .append("\nResponse: ").append(response)
                      .append("\n")
                      .toString();
              System.out.println(req);

              // Return the response:
              return new JSONObject(response);
          }

          // Create the main function to form and send signed requests:
          public static void main(String[] args) throws Exception {
              // Generate a key pair:
              Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());
              SecureRandom secureRandom = new SecureRandom();
              ECGenParameterSpec spec = new ECGenParameterSpec("secp256k1");
              KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("ECDSA");
              keyPairGenerator.initialize(spec, secureRandom);
              KeyPair keyPair = keyPairGenerator.generateKeyPair();

              // Convert the public key into PEM format:
              ByteArrayOutputStream baos = new ByteArrayOutputStream();
              JcaPEMWriter jcaPEMWriter = new JcaPEMWriter(new OutputStreamWriter(baos, StandardCharsets.UTF_8));
              jcaPEMWriter.writeObject(keyPair.getPublic());
              jcaPEMWriter.flush();
              jcaPEMWriter.close();
              String publicKey = new String(baos.toByteArray());

              // The private key is required to sign requests.
              // Convert it into PEM format and make sure to put into a file to save it in some secure place later:
              StringWriter stringWriter = new StringWriter();
              JcaPEMWriter pemWriter = new JcaPEMWriter(stringWriter);
              try (PrintStream out = new PrintStream(new FileOutputStream("private.pem"))) {
                  pemWriter.writeObject(keyPair);
                  pemWriter.close();
                  String pem = stringWriter.toString();
                  out.print(pem);
              }

              // Get a seed and cookie to form a request:
              Pair seedAndCookie = getNewSeed();
              String seed = seedAndCookie.getSeed();
              String cookie = seedAndCookie.getCookie();
              // Form a request body for member.create:
              Schema.Params memberParams = new Schema.Params();
              memberParams.setSeed(seed);
              memberParams.setCallSite("member.create");
              memberParams.setPublicKey(publicKey);

              // Form a JSON payload:
              String createMemberReq = new Schema.requestBody().withMethod("contract.call").withParams(memberParams).withID(id).toJson();

              // Increment the JSON RPC 2.0 request identifier for future requests:
              id++;

              // Send the signed member.create request and pass the cookie:
              JSONObject newMember = sendSignedRequest(createMemberReq, keyPair.getPrivate(), cookie);
              assert newMember.isNull("error");

              // Put the reference to your new member into a variable to send subsequent transfer requests:
              String memberReference = newMember.getJSONObject("result").getJSONObject("callResult").getString("reference");
              System.out.println("Member reference is " + memberReference);

              // Get a new seed to form a transfer request:
              seedAndCookie = getNewSeed();
              seed = seedAndCookie.getSeed();
              cookie = seedAndCookie.getCookie();

              // Form a request body for transfer:
              Schema.Params transferParams = new Schema.Params();
              transferParams.setSeed(seed);
              transferParams.setCallSite("member.transfer");
              transferParams.setPublicKey(publicKey);
              transferParams.setCallParams(new Schema.TransferCallParams("100", "<recipient_member_reference>"));
              transferParams.setReference(memberReference);

              // Form a JSON payload:
              String transferReq = new Schema.requestBody().withMethod("contract.call").withParams(transferParams).withID(id).toJson();

              // Increment the id for future requests:
              id++;

              // Send the signed transfer request:
              JSONObject newTransfer = sendSignedRequest(transferReq, keyPair.getPrivate(), cookie);
              assert newTransfer.isNull("error");
              String fee = newTransfer.getJSONObject("result").getJSONObject("callResult").getString("fee");

              // (Optional) Print out the fee.
              System.out.println("Fee is " + fee);
          }
      }

|
