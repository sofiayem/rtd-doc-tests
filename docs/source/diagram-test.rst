Components in the diagram are *clickable*, the links will lead you to respective definitions.

.. uml::
   
   scale 750 width
   together {
   package "Pulsar consensus" as pcon [[../architecture.html#pulsar-consensus]] {
      collections Pulsars [[../architecture.html#pulsars]]
   }
   package "Cloud n+1" as cloudnext [[../architecture.html#fed-of-clouds]] {
      card [Cloud's network consensus\n & messaging] as nextnet [[../architecture.html#network-consensus]]
      frame "Domain b" as domb [[../architecture.html#domains]] {
         rectangle "Contract b" as cntrctb [[../architecture.html#contracts]]
      }
      frame "Domain a" as doma [[../architecture.html#domains ]] {
         rectangle "Contract a" as cntrcta [[../architecture.html#contracts]]
      }
      cntrctb -[hidden]d- nextnet
      cntrcta -[hidden]d- nextnet
   Pulsars -u-> nextnet : pulses
   }
   }
   package "Cloud n" as cloudn [[../architecture.html#fed-of-clouds]] {
      frame "Domain n" as domn [[../architecture.html#domains]] {
        rectangle "Contract n+1" as cntrctnext [[../architecture.html#contracts]]
        rectangle "Contract n" as cntrctne [[../architecture.html#contracts]]
      }
      rectangle "Network globulas" as globula [[../architecture.html#globulas]] {
        node "Virtual nodes" as vn [[../architecture.html#virtual]] {
            card vcard [
              - Generation handling
              - Transacting
              - CPU scaling
            ]
        }
        node "Light material nodes" as ln [[../architecture.html#light-material]] {
            card lcard [
              - Short-term storage
              - Traffic scaling
              - Block Building
            ]
        }
        node "Heavy material nodes" as hn [[../architecture.html#heavy-material]] {
            card hcard [
              - Long-term storage
              - Replication & recovery
              - Storage scaling
            ]
        }
      }
      together {
      card [Cloud's network consensus\n & messaging] as net [[../architecture.html#network-consensus]]
      database "Ledger" as db [[../architecture.html#ledger]] {
        frame "Storage, validation & consensus" [[../architecture.html#storage-consensus]] {
        rectangle ldgr [
          - Permissions
          ....
          - Integrity & replication
          ....
          - Jets, lifelines & records
        ]
        }
      }
      node "Processing" as process [[../architecture.html#execution-validation]] {
        frame "Logic validation & consensus" [[../architecture.html#logic-consensus]] {
        rectangle proc [
          - Compilers
          ....
          - Artifact cache 
          - Security context
          ....
          - Distributed transaction
            management
        ]
        }
      }
      }
      domn -[hidden]- globula
      vcard -[hidden]d- lcard
      lcard -[hidden]d- hcard
      net -[hidden]d- process
      proc <-d-> net : data & code
      net <-d-> ldgr : data & code
      net -[hidden]r- ln
      db -[hidden]r- hn
      process -[hidden]d- net
      proc -[hidden]r- vn
      Pulsars -r-> net: pulses
      net <-> nextnet : messages
      domb -[hidden]- net
      domb -[hidden]r- domn
      domb -[hidden]r- proc
      domb -[hidden]r- net
   }

Diagram test.