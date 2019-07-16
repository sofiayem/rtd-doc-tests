.. Insolar documentation master file, created by
   sphinx-quickstart on Tue May 14 19:35:14 2019.

Insolar Documentation
=====================

.. note::

   Insolar's documentation is under development, its structure is not yet finalized. Expect updates soon.

To get a grip on how Insolar works, take a look at its :ref:`architecture overview <architecture>`.

To connect to TestNet 1.1, go through :ref:`step-by-step instructions <connecting_to_testnet>`.

.. // TODO: Develop a documentation entry point (main page) upon the content's readiness.

Tabs extension:

.. tabs::

   .. tab:: Python

      .. code-block:: python

         import my-api

   .. tab:: Ruby

      .. code-block:: ruby

         require 'my-api'

   .. tab:: Golang

      .. code-block:: guess

         import {'smth'}

.. content-tabs::

   .. tab-container:: tab1
      :title: Golang

      .. code-block:: guess

         import {'smth'}

   .. tab-container:: tab2
      :title: Java

      .. toggle-header::
         :header: Java code **Show/Hide**

         .. code-block:: guess

            import {'smth else'}

.. toggle-header::
   :header: Example 1 **Show/Hide Code**

   .. code-block:: go

      // Create a function to get a new seed for each signed request:
      func getNewSeed() string {
        // Form a request body for getSeed:
        getSeedReq := platformRequest{
          JSONRPC: JSONRPCVersion,
          Method:  "node.getSeed",
          ID:      id,
        }
        // Increment the id for future requests:
        id++

        // Send the request:
        seedBody := sendInfoRequest(getSeedReq)

        // Unmarshal the response:
        var seed seedResponse
        err := json.Unmarshal(seedBody, &seed)
        if err != nil {
          log.Fatalln(err)
        }
        // Put the current seed into a variable:
        return seed.Result.Seed
      }

.. code-block:: go

   package main

   import (
      // We'll need:
      // - Some basic Golang functionality.
      "bytes"
      "io/ioutil"
      "fmt"
      "log"
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

   // Create a variable for the JSON RPC 2.0 request identifier:
   var id int = 1
   // The identifier is to be incremented in every request and each response will contain a corresponding one.

   // Define constants the requests will use:
   const (
      // Some HTTP parameters:
      Digest         = "Digest"
      Signature      = "Signature"
      ContentType    = "Content-Type"
      appjs          = "application/json"
      // JSON RPC protocol version:
      JSONRPCVersion     = "2.0"
      // Endpoint URL for local deployment (is to be changed to a production URL):
      url            = "http://127.0.0.1:19101/api/"
   )

   // Create and initialize an HTTP client for connection re-use:
   var client *http.Client

   func init() {
      client = &http.Client{}
   }

   // Declare a structure to contain the ECDSA signature:
   type ecdsaSignature struct {
      R, S *big.Int
   }

   // Declare a nested structure to form requests to Insolar's API in accordance with the specification.
   // The Platform uses the basic JSON RPC 2.0 request structure:
   type platformRequest struct {
      JSONRPC        string         `json:"jsonrpc"`
      ID             int            `json:"id"`
      Method         string         `json:"method"`
      Params         platformParams `json:"params"`
   }

   // The Platform defines params of the signed request as follows:
   type platformParams struct {
      Seed            string    `json:"seed"`
      CallSite        string    `json:"callSite"`
      CallParams      interface{}  `json:"callParams"`
      Reference       string    `json:"reference"`
      PublicKey string    `json:"memberPublicKey"`
   }
   // Where callParams is a structure that depends on a particular method:
   type memberCreateParams struct {}

   type transferParams struct {
      Amount            string    `json:"amount"`
      ToMemberReference string    `json:"toMemberReference"`
   }

   // Declare structures to unmarshal the responses into in accordance with the specification.
   // The Platform uses the basic JSON RPC 2.0 response structure:
   type platformResponse struct{
      JSONRPC      string    `json:"jsonrpc"`
      ID           int       `json:"id"`
   }

   // The result of the seed request is as follows:
   type seedResponse struct {
      platformResponse
      Result seedResult `json:"result"`
   }

   type seedResult struct {
      Seed     string    `json:"Seed"`
      TraceID  string    `json:"TraceID"`
   }

   // The result of the info requests is as follows:
   type infoResponse struct {
      platformResponse
      Result infoResult `json:"result"`
   }

   type infoResult struct {
      RootDomain                 string   `json:"RootDomain"`
      RootMember                 string   `json:"RootMember"`
      MigrationAdminMember       string   `json:"MigrationAdminMember"`
      MigrationDaemonMembers []  string   `json:"MigrationDaemonMembers"`
      NodeDomain                 string   `json:"NodeDomain"`
      TraceID                    string   `json:"TraceID"`
   }

   // The result of the member.create request is as follows:
   type memberResponse struct {
      platformResponse
      Result memberResult
   }
   type memberResult struct {
      CallResult     struct {
         Reference  string    `json:"reference"`
      }                        `json:"callResult"`
      TraceID        string    `json:"TraceID"`
   }

   // Create a function to send information requests:
   func sendInfoRequest(payload platformRequest) []byte {
      // Marshal the payload into JSON:
      jsonPayload, err := json.Marshal(payload)
      if err != nil {
         log.Fatalln(err)
      }

      // Create a new HTTP request and send it:
      request, err := http.NewRequest("POST", url+"rpc", bytes.NewBuffer(jsonPayload))
      if err != nil {
         log.Fatalln(err)
      }
      request.Header.Set(ContentType, appjs)
      response, err := client.Do(request)
      if err != nil {
         log.Fatalln(err)
      }
      defer request.Body.Close()

      // Receive and return the response body:
      responseBody, err := ioutil.ReadAll(response.Body)
      if err != nil {
         log.Fatalln(err)
      }
      fmt.Println(string(responseBody))
      return responseBody
   }

   // Create a function to send signed requests:
   func sendSignedRequest(payload platformRequest, privateKey *ecdsa.PrivateKey) []byte {
      // Marshal the payload into JSON:
      jsonPayload, err := json.Marshal(payload)
      if err != nil {
         log.Fatalln(err)
      }

      // Take a SHA-256 hash of the payload:
       hash := sha256.Sum256(jsonPayload)

       // Sign the hash with the private key:
       r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:])
      if err != nil {
         log.Fatalln(err)
      }

       // See if the signature is valid:
       valid := ecdsa.Verify(&privateKey.PublicKey, hash[:], r, s)
      fmt.Println("signature verified:", valid)

      // Convert the signature into ASN.1 format:
      sig := ecdsaSignature{
         R: r,
         S: s,
      }
      signature, _ := asn1.Marshal(sig)

       // Convert both hash and signature into a Base64 string:
      hash64 := base64.StdEncoding.EncodeToString(hash[:])
       signature64 := base64.StdEncoding.EncodeToString(signature)

       // Set headers and send the signed request:
      request, err := http.NewRequest("POST", url+"call", bytes.NewBuffer(jsonPayload))
      if err != nil {
         log.Fatalln(err)
      }
      request.Header.Set(ContentType, appjs)

      // Put the hash string into the HTTP Digest header:
      request.Header.Set(Digest, "SHA-256="+hash64)

      // Put the signature string into the HTTP Signature header:
      request.Header.Set(Signature, "keyId=\"member-pub-key\", algorithm=\"ecdsa\", headers=\"digest\", signature="+signature64)
      fmt.Println(request.Header)
      fmt.Println(request.Body)

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
       return responseBody
   }

   // Create an instance of the seed response structure to unmarshal every new seed in:
   var seed seedResponse

   // Create a function to get a new seed for each signed request:
   func getNewSeed() string {
      // Form a request body for getSeed:
      getSeedReq := platformRequest{
         JSONRPC: JSONRPCVersion,
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
      seedReq.Header.Set(ContentType, appjs)
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
      fmt.Println(string(seedRespBody))

      // Unmarshal the response:
      err = json.Unmarshal(seedRespBody, &seed)
      if err != nil {
         log.Fatalln(err)
      }
      // Put the current seed into a variable:
      return seed.Result.Seed
   }

   func main() {

      // Form a request body for getInfo:
      getInfoReq := platformRequest{
         JSONRPC: JSONRPCVersion,
         Method:  "network.getInfo",
         ID:      id,
      }
      // Increment the id for future requests:
      id++

      // Send the request:
      infoBody := sendInfoRequest(getInfoReq)

      // Unmarshal the response:
      var info infoResponse
      err := json.Unmarshal(infoBody, &info)
      if err != nil {
         log.Fatalln(err)
      }
      // Put the rootMember reference into a variable:
      rootMember := info.Result.RootMember

      // Create a key pair:
      privateKey := new(ecdsa.PrivateKey)
      privateKey, err = ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
       var publicKey ecdsa.PublicKey
       publicKey = privateKey.PublicKey

       // Convert the public key into PEM format:
       x509PublicKey, err := x509.MarshalPKIXPublicKey(&publicKey)
      if err != nil {
         log.Fatalln(err)
      }
      pemPublicKey := pem.EncodeToMemory(&pem.Block{Type: "PUBLIC KEY", Bytes: x509PublicKey})

      // Form a request body for member.create:
      seed := getNewSeed()
      createMemberReq := platformRequest{
         JSONRPC: JSONRPCVersion,
         Method:  "api.call",
         ID:      id,
         Params:platformParams {
            Seed: seed,
            CallSite: "member.create",
            CallParams:memberCreateParams {},
            Reference: string(rootMember),
            PublicKey: string(pemPublicKey),},
      }
      // Increment the id for future requests:
      id++

      // Send the signed member.create request:
      newMember := sendSignedRequest(createMemberReq, privateKey)
      fmt.Println(string(newMember))
       var member memberResponse
      err = json.Unmarshal(newMember, &member)
      if err != nil {
         log.Fatalln(err)
      }
      // Put your reference into a variable to form a transfer request next:
      memberReference := member.Result.CallResult.Reference
      fmt.Println(memberReference)

       // Get a new seed and form a transfer request:
      seed = getNewSeed()

      // Form a request body for transfer:
      transferReq := platformRequest{
         JSONRPC: JSONRPCVersion,
         Method:  "api.call",
         ID:      id,
         Params:platformParams {
            Seed: seed,
            CallSite: "wallet.transfer",
            CallParams:transferParams {
               Amount: "100",
               ToMemberReference: "7PsQanNDwG1y3hWZr3FqEUs624Js1ASR1seQ2VeV23Y.11111111111111111111111111111111",
            },
            Reference: string(memberReference),
            PublicKey: string(pemPublicKey),},
      }
      // Send the signed transfer request:
      newTransfer := sendSignedRequest(transferReq, privateKey)
      fmt.Println(string(newTransfer))
   }

.. toctree::
   :maxdepth: 2
   :caption: Overview

   about
   features
   architecture
   glossary

.. // "Integration" below is a [WIP] title later to be expanded into a collection of how-to guides and renamed appropriately (if required).
 
.. toctree::
   :maxdepth: 2
   :caption: Integration

   integration
