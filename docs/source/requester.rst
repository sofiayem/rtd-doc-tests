.. _building_requester:

===================================
Tutorial: Building an API Requester
===================================

This tutorial walks you through the process of creating a requester to Insolar’s API in which you will learn how to form and sign requests that create a member capable of transferring funds to other members.

To start building the requester, first, familiarize yourself with definitions Insolar uses.

.. _definitions:

Insolar Definitions
-------------------

Insolar is being developed to provide inter-operation between enterprises.

From a business perspective:

* Any enterprise is regarded as an *entity* with a corresponding *identity* -- a **member** object stored in the blockchain.
* Any member has its own *account* to/from which the member can transfer funds.

From a technical perspective:

* Every member object is identified by a **reference** in the blockchain.
* The reference is effectively the member's *address*.

Therefore, an entity wishing to transfer funds to/from its account from/to a particular address, in Insolar's terms, is a member object identified by its reference wishing to transfer funds to some other member identified by the corresponding reference.

Insolar uses these terms in method and parameter names.

.. _what_you_will_build:

What You Will Build
-------------------

You will build a program (requester) that creates a member in the Insolar network and, as a member, transfers funds to and from its account.

The requester forms and sends the following requests to the Insolar’s JSON RPC 2.0 API endpoint and receives the corresponding responses:

#. Gets a **seed** that grants permission to call a contract’s method.

#. Forms, signs, and sends a **create-member** request with the seed and receives your member’s reference in response.

#. Forms, signs, and sends a **transfer** request with another seed that:

   * sends an amount of Insolar coins (XNS) from your member’s account to another’s;
   * given your reference received in the previous response and a reference to the recipient's member.

.. _what_you_will_need:

What You Will Need
------------------

* About half an hour.
* Your favorite IDE for:

  * either Golang,
  * or Java.

.. _how_to_complete:

How to Complete This Tutorial
-----------------------------

* To start from scratch, go through the :ref:`step-by-instructions <build_requester>` listed below.
* To skip the basics, read (and copy-paste) the working :ref:`requester code <requester_example>` provided at the end.

.. _build_requester:

Building the Requester
----------------------

To build the requester, go through the following steps:

#. **Prepare**: install the necessary tools, set up a local Insolar deployment, import the necessary packages, and initialize an HTTP client.

#. **Declare** request **structures** (Golang) or **classes** (Java) in accordance with the Insolar’s API specification.

#. **Create a seed getter** function. This getter will be reused to put a unique seed into every signed request.

#. **Generate a key pair**, export the private key into a file, and store it in some secure place.

#. **Create a sender** function that signs and sends requests given the private key.

#. **Form a member-create request** and call the sender function to send it.

#. **Form a transfer request** and, again, call the sender function.

All the above steps are detailed in sections below.

.. _prepare:

Step 1: Prepare
~~~~~~~~~~~~~~~

To build the requester, you will need the following:

#. Set up your development environment if you do not have one. Install one of the following:

   * `Go programming tools <https://golang.org/doc/install>`_;
   * `Java programming tools <https://java.com/en/download/help/download_options.xml>`_.

#. Set up an :ref:`Insolar network locally <setting_up_devnet>` to test your future requester against.

#. (Only Java) Create a ``pom.xml`` file to contain information about the project and configuration details used by Maven to build it:

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

   |

#. With the Golang or Java programming tools, you do not need to “reinvent the wheel”: create a ``main.go`` or ``main.java`` file and, inside, import the necessary packages listed below.

   .. content-tabs::

      .. tab-container:: Golang
         :title: Golang: Main.go

         .. code-block:: Go
            :linenos:

            package main

            import (
               // We'll need:
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

            // We'll need:
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

      * (Golang) Since an ECDSA signature in Go consists of two big integers, declare a single structure to contain it.
      * (Java) Since the program has to contain a main class, declare it to wrap up all the required functionality.

   #. Set the endpoint URL for local deployment. It can be changed to a production one after testing.
   #. Create and initialize an HTTP client for connection re-use.
   #. Create a variable for the JSON RPC 2.0 request identifier. The identifier is to be incremented for every request and each corresponding response will contain it.

   For example:

   .. content-tabs::

      .. tab-container:: Golang
         :title: Golang: Main.go

         .. code-block:: Go
            :lineno-start: 28

            // Declare a structure to contain the ECDSA signature:
            type ecdsaSignature struct {
               R, S *big.Int
            }

            // Set the endpoint URL for local deployment (is to be changed to a production URL):
            const (
               url = "http://127.0.0.1:19101/api/"
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
            :lineno-start: 24

            // Declare a main class to wrap up all the required functionality:
            public class Main {

                // Set the endpoint URL for local deployment (is to be changed to a production URL):
                private static final String API_URL = "http://localhost:19101/api";

                // Create and initialize an HTTP client for connection re-use:
                private static final HttpClient client = HttpClient.newBuilder().build();

                // Create a variable for the JSON RPC 2.0 request identifier:
                static Integer id = 1;
                // The identifier is to be incremented for every request and each corresponding response will contain it.

                // The Main class is to be continued...

With that, everything your requester will need is set up.

.. _declare_structs_or_classes:

Step 2: Declare Request Structures or Classes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, declare request structures (Golang) or classes (Java) in accordance with the Insolar’s API specification:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. code-block:: Go
         :lineno-start: 48

         // Continue in the Main.go file...

         // Declare a nested structure to form requests to Insolar's API in accordance with the specification.
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

         // Declare structures to unmarshal the seed response in accordance with the specification.
         // The Platform uses the basic JSON RPC 2.0 response structure:
         type responseBody struct{
            JSONRPC      string    `json:"jsonrpc"`
            ID           int       `json:"id"`
         }

         // The result of the seed request is as follows:
         type seedResponse struct {
            responseBody
            Result seedResult `json:"result"`
         }

         type seedResult struct {
            Seed     string    `json:"seed"`
            TraceID  string    `json:"traceID"`
         }

   .. tab-container:: Java
      :title: Java: Main.java

      .. hint:: In Java, create the corresponding setters to initialize class instances later.

      .. code-block:: Java
         :lineno-start: 38

         // Continue in the Main class...

         // Declare a class to build a request:
         public static class Schema {

           // Declare a class to form requests to Insolar's API in accordance with the specification.
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
               // CallParams is a structure that depends on a particular method.
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

.. _create_seed_getter:

Step 3: Create a Seed Getter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each signed request to Insolar's API has to contain a seed in its body. Seed is a unique piece of information that:

   * grants permission to send a request;
   * has a short lifespan;
   * expires upon first use.

To be able to send signed requests, create a seed getter function to re-use upon forming each such request:

.. content-tabs::

   .. tab-container:: Golang
      :title: Golang: Main.go

      .. code-block:: Go
         :lineno-start: 100

         // Continue in the Main.go file...

         func getNewSeed() string {
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
             seedReq, err := http.NewRequest("POST", url+"rpc", bytes.NewBuffer(jsonSeedReq))
             if err != nil {
                log.Fatalln(err)
             }
             seedReq.Header.Set("ContentType", "application/json")
             seedResponse, err := client.Do(seedReq)
             if err != nil {
                log.Fatalln(err)
             }
             defer seedReq.Body.Close()

             // Receive and return the response body:
             seedRespBody, err := ioutil.ReadAll(seedResponse.Body)
             if err != nil {
                log.Fatalln(err)
             }

             // Unmarshal the response:
             err = json.Unmarshal(seedRespBody, &seed)
             if err != nil {
                log.Fatalln(err)
             }

             // (Optional) Print the request and its response:
             print := "POST to " + url+"call" +
             "\nPayload: " + string(jsonSeedReq) +
             "\nResponse status code: " +  strconv.Itoa(seedResponse.StatusCode) +
             "\nResponse: " + string(seedRespBody) + "\n"
             fmt.Println(print)

             // Return the current seed:
             return seed.Result.Seed
          }

   .. tab-container:: Java
      :title: Java: Main.java

      .. code-block:: Java
         :lineno-start: 133

         // Continue in the Main class...

         // Create a function to get a new seed for each signed request:
         private static String getNewSeed() throws Exception {
           // Form a request body for getSeed and format it into JSON:
           String seedRequest = new Schema.requestBody().withMethod("node.getSeed").withID(id).toJson();
           // Increment the id for future requests:
           id++;

           // Create a new HTTP request and send it:
           URL url = new URL(API_URL.concat("/rpc"));
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
                   .append("\nResponse status code = ").append(send.statusCode())
                   .append("\nResponse: ").append(response)
                   .append("\n")
                   .toString();
           System.out.println(req);

           // Return the current seed:
           return new JSONObject(response).getJSONObject("result").getString("Seed");
         }

.. _generate_key_pair:

Step 4: Generate a Key Pair
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The body of each request that calls a contract's method must be hashed by a ``SHA256`` algorithm. Each hash must be signed by a private key generated by a ``P256`` (Golang) or ``secp256k1`` (Java) elliptic curve.

To be able to sign requests, do the following:

#. Generate a key pair using the said curve.
#. Export the private key into a file.
#. Save the file into some secure place.

.. warning:: You will not be able to access your member object without the private key and, as such, transfer funds.

For example: 

to be continued... 

.. _requester_example:

Complete Requester Code Examples
--------------------------------

.. toggle-header::
   :header: Golang code **Show/Hide**

   .. code-block:: Go
      :linenos:

      package main

      import (
         // We'll need:
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

      // Set the endpoint URL for local deployment (is to be changed to a production URL):
      const (
         url = "http://127.0.0.1:19101/api/"
      )

      // Create and initialize an HTTP client for connection re-use:
      var client *http.Client

      func init() {
         client = &http.Client{}
      }

      // Create a variable for the JSON RPC 2.0 request identifier:
      var id int = 1
      // The identifier is to be incremented for every request and each corresponding response will contain it.

      // Declare a nested structure to form requests to Insolar's API in accordance with the specification.
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

      // Declare structures to unmarshal the seed response in accordance with the specification.
      // The Platform uses the basic JSON RPC 2.0 response structure:
      type responseBody struct{
         JSONRPC      string    `json:"jsonrpc"`
         ID           int       `json:"id"`
      }

      // The result of the seed request is as follows:
      type seedResponse struct {
         responseBody
         Result seedResult `json:"result"`
      }

      type seedResult struct {
         Seed     string    `json:"seed"`
         TraceID  string    `json:"traceID"`
      }

      // Create an instance of the seed response structure to unmarshal every new seed in:
      var seed seedResponse

      // Create a function to get a new seed for each signed request:
      func getNewSeed() string {
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
         seedReq, err := http.NewRequest("POST", url+"rpc", bytes.NewBuffer(jsonSeedReq))
         if err != nil {
            log.Fatalln(err)
         }
         seedReq.Header.Set("ContentType", "application/json")
         seedResponse, err := client.Do(seedReq)
         if err != nil {
            log.Fatalln(err)
         }
         defer seedReq.Body.Close()

         // Receive and return the response body:
         seedRespBody, err := ioutil.ReadAll(seedResponse.Body)
         if err != nil {
            log.Fatalln(err)
         }

         // Unmarshal the response:
         err = json.Unmarshal(seedRespBody, &seed)
         if err != nil {
            log.Fatalln(err)
         }

         // (Optional) Print the request and its response:
         print := "POST to " + url+"call" +
         "\nPayload: " + string(jsonSeedReq) +
         "\nResponse status code: " +  strconv.Itoa(seedResponse.StatusCode) +
         "\nResponse: " + string(seedRespBody) + "\n"
         fmt.Println(print)

         // Return the current seed:
         return seed.Result.Seed
      }

      // Create a function to send signed requests:
      func sendSignedRequest(payload requestBody, privateKey *ecdsa.PrivateKey) map[string]interface{} {
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
         request, err := http.NewRequest("POST", url+"call", bytes.NewBuffer(jsonPayload))
         if err != nil {
            log.Fatalln(err)
         }
         request.Header.Set("ContentType", "application/json")

         // Put the hash string into the HTTP Digest header:
         request.Header.Set("Digest", "SHA-256="+hash64)

         // Put the signature string into the HTTP Signature header:
         request.Header.Set("Signature", "keyId=\"public-key\", algorithm=\"ecdsa\", headers=\"digest\", signature="+signature64)

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

         // Print the request and its response:
         print := "POST to " + url+"call" +
         "\nPayload: " + string(jsonPayload) +
         "\nResponse status code: " + strconv.Itoa(response.StatusCode) +
         "\nResponse: " + string(responseBody) + "\n"
         fmt.Println(print)
         return JSONObject
      }

      func main() {
         // Create a key pair:
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

         // The private key is necessary to sign requests.
         // Make sure to put into a file to save it in some secure place later:
         file, err := os.Create("private.pem")
          if err != nil {
              fmt.Println(err)
              return
          }
          file.WriteString(string(pemPrivateKey))
          file.Close()

         // Form a request body for member.create:
         seed := getNewSeed()
         createMemberReq := requestBody{
            JSONRPC: "2.0",
            Method:  "api.call",
            ID:      id,
            Params:params {
               Seed: seed,
               CallSite: "member.create",
               CallParams:memberCreateCallParams {},
      //       Reference: "",
               PublicKey: string(pemPublicKey),},
         }
         // Increment the id for future requests:
         id++

         // Send the signed member.create request:
         newMember := sendSignedRequest(createMemberReq, privateKey)

         // Put the reference to your new member into a variable to send transfer requests:
         memberReference := newMember["result"].(map[string]interface{})["callResult"].(map[string]interface{})["reference"].(string)
         fmt.Println("Member reference is " + memberReference)
         // Get a new seed to form a transfer request:
         seed = getNewSeed()

         // Form a request body for transfer:
         transferReq := requestBody{
            JSONRPC: "2.0",
            Method:  "api.call",
            ID:      id,
            Params:paramsWithReference{ params:params{
                  Seed: seed,
                  CallSite: "member.transfer",
                  CallParams:transferCallParams {
                     Amount: "100",
                     ToMemberReference: "7ZB6H4SLyiRW6LnMmFkeGMgWreoaWfCsHtFMr7s3E7t.11111111111111111111111111111111",
                  },
                  PublicKey: string(pemPublicKey),
               },
               Reference: string(memberReference),
            },
         }
         // Increment the id for future requests:
         id++

         // Send the signed transfer request:
         newTransfer := sendSignedRequest(transferReq, privateKey)
         fee := newTransfer["result"].(map[string]interface{})["callResult"].(map[string]interface{})["fee"].(string)
         fmt.Println("Fee is " + fee)
      }

|

.. toggle-header::
   :header: Java code **Show/Hide**

   .. code-block:: Java
      :linenos:

      package requester;

      // We'll need:
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

      // Declare a main class to wrap up all the required functionality:
      public class Main {

          // Set the endpoint URL for local deployment (is to be changed to a production URL):
          private static final String API_URL = "http://localhost:19101/api";

          // Create and initialize an HTTP client for connection re-use:
          private static final HttpClient client = HttpClient.newBuilder().build();

          // Create a variable for the JSON RPC 2.0 request identifier:
          static Integer id = 1;
          // The identifier is to be incremented for every request and each corresponding response will contain it.

          // Declare a class to build a request:
          public static class Schema {

              // Declare a class to form requests to Insolar's API in accordance with the specification.
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
                  // CallParams is a structure that depends on a particular method.
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

          // Create a function to get a new seed for each signed request:
          private static String getNewSeed() throws Exception {
              // Form a request body for getSeed and format it into JSON:
              String seedRequest = new Schema.requestBody().withMethod("node.getSeed").withID(id).toJson();
              // Increment the id for future requests:
              id++;

              // Create a new HTTP request and send it:
              URL url = new URL(API_URL.concat("/rpc"));
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
                      .append("\nResponse status code = ").append(send.statusCode())
                      .append("\nResponse: ").append(response)
                      .append("\n")
                      .toString();
              System.out.println(req);

              // Return the current seed:
              return new JSONObject(response).getJSONObject("result").getString("Seed");
          }

          // Create a function to send signed requests:
          private static JSONObject sendSignedRequest(String requestBody, PrivateKey privateKey) throws Exception {

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

              // Create a new request and send it:
              URL url = new URL(API_URL.concat("/call"));
              HttpRequest request = HttpRequest.newBuilder()
                      .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                      .header("Content-Type", "application/json; utf-8")
                      .header("Digest", digestHeader)
                      .header("Signature", signatureHeader)
                      .uri(url.toURI())
                      .build();
              HttpResponse<String> send = client.send(request, HttpResponse.BodyHandlers.ofString());

              assert send.statusCode() == 200;

              // Receive the response:
              String response = send.body();

              // Print the request and its response:
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

          public static void main(String[] args) throws Exception {
              // Create a key pair:
              Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());
              SecureRandom secureRandom = new SecureRandom();
              ECGenParameterSpec spec = new ECGenParameterSpec("secp256k1");
              KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("ECDSA");
              keyPairGenerator.initialize(spec, secureRandom);
              KeyPair keyPair = keyPairGenerator.generateKeyPair();

              // The private key is necessary to sign requests.
              // Make sure to put into a file to save it in some secure place later:
              StringWriter stringWriter = new StringWriter();
              JcaPEMWriter pemWriter = new JcaPEMWriter(stringWriter);
              try (PrintStream out = new PrintStream(new FileOutputStream("private.pem"))) {
                  pemWriter.writeObject(keyPair);
                  pemWriter.close();
                  String pem = stringWriter.toString();
                  out.print(pem);
              }

              // Convert the public key into PEM format:
              ByteArrayOutputStream baos = new ByteArrayOutputStream();
              JcaPEMWriter jcaPEMWriter = new JcaPEMWriter(new OutputStreamWriter(baos, StandardCharsets.UTF_8));
              jcaPEMWriter.writeObject(keyPair.getPublic());
              jcaPEMWriter.flush();
              jcaPEMWriter.close();
              String publicKey = new String(baos.toByteArray());

              // Form a request body for member.create:
              String seed = getNewSeed();
              Schema.Params memberParams = new Schema.Params();
              memberParams.setSeed(seed);
              memberParams.setCallSite("member.create");
              memberParams.setPublicKey(publicKey);

              // Form a JSON payload:
              String createMemberReq = new Schema.requestBody().withMethod("api.call").withParams(memberParams).withID(id).toJson();

              // Increment the id for future requests:
              id++;

              // Send the signed member.create request:
              JSONObject newMember = sendSignedRequest(createMemberReq, keyPair.getPrivate());
              assert newMember.isNull("error");

              // Put the reference to your new member into a variable to send transfer requests:
              String memberReference = newMember.getJSONObject("result").getJSONObject("callResult").getString("reference");
              System.out.println("Member reference is " + memberReference);

              // Get a new seed to form a transfer request:
              seed = getNewSeed();

              // Form a request body for transfer:
              Schema.Params transferParams = new Schema.Params();
              transferParams.setSeed(seed);
              transferParams.setCallSite("member.transfer");
              transferParams.setPublicKey(publicKey);
              transferParams.setCallParams(new Schema.TransferCallParams("100", "7ZB6H4SLyiRW6LnMmFkeGMgWreoaWfCsHtFMr7s3E7t.11111111111111111111111111111111"));
              transferParams.setReference(memberReference);

              // Form a JSON payload:
              String transferReq = new Schema.requestBody().withMethod("api.call").withParams(transferParams).withID(id).toJson();

              // Increment the id for future requests:
              id++;

              // Send the signed transfer request:
              JSONObject newTransfer = sendSignedRequest(transferReq, keyPair.getPrivate());
              assert newTransfer.isNull("error");
              String fee = newTransfer.getJSONObject("result").getJSONObject("callResult").getString("fee");
              System.out.println("Fee is " + fee);
          }
      }

|

