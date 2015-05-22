slidenumbers: true

# [fit] Architecting for
# [fit] Continuous Delivery
# [fit] with
# [fit] Microservices
![](https://raw.githubusercontent.com/spring-projects/spring-cloud/gh-pages/img/project-icon-large.png)

---

# [fit] Session
# [fit] Three
![](Common/images/cf_logo.png)

---

# [fit] Testing  
# [fit] and Delivering
# [fit] Microservices

---

# [fit] _Bounded_
# [fit] Contexts

---

# [fit] _CONWAY'S_
# [fit] LAW

---

# Conway's Law

> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
-- Melvyn Conway, 1967

---

# The Inverse Conway Manuever
![inline fit](Common/images/inverse_conway.png)

---

# [fit] Decentralized
# [fit] _Autonomous_
# [fit] Capability
# [fit] _Teams / Services_

---

# [fit] TESTING

---

# [fit] Consumer
# [fit] Driven
# [fit] _Contracts_

---

# Bounded Contexts

- _Crossing Boundaries_ :arrow_right: _Increased Complexity_
- _Interfaces to Other Contexts_
- _Availability of Other Services_

---

# [fit] _THIS_
# [fit] _IS_
# [fit] HARD

---

# [fit] _Consumer to Component_ =
# [fit] CONTRACT

---

# [fit] Contracts

- _Input/Output Data Structure Expectations_
- _Side Effects_
- _Performance Characteristics_
- _Concurrency Characteristics_

---

# [fit] _MULTIPLE CONSUMERS_
# [fit] =
# [fit] _MULTIPLE CONTRACTS_

---

# [fit] _Contracts_
# [fit] Must Be Met Over :alarm_clock:

---

# [fit] _API_ =
# [fit] Contract

---

# _Consumer-Driven_ Contract

_I capture my_ expectations _of your service as a suite of automated_ tests _that I_ provide _to you._

_You_ run _them in your_ continuous integration _pipeline._

---

# [fit] Remember
# [fit] _Decentralized_
# [fit] _Autonomous_
# [fit] _Capability_
# [fit] _Teams/Services_?

---

# [fit] _Consumer-Driven_ Contracts
# :thumbsup:

---

# [fit] _PACT_
https://github.com/realestate-com-au/pact

---

# [fit] PACT _JVM_
https://github.com/DiUS/pact-jvm

---

# Consumer `PactFragment`

```java
@Pact(state = "FortuneState", provider = "FortuneService", consumer = "FortuneUi")
public PactFragment createFragment(ConsumerPactBuilder.PactDslWithProvider.PactDslWithState builder) {
    Map<String, String> headers = new HashMap<>();
    headers.put("Content-Type", "application/json;charset=UTF-8");

    PactDslJsonBody responseBody = new PactDslJsonBody()
            .numberType("id")
            .stringType("text");

    return builder.uponReceiving("a request for a random fortune")
            .path("/random")
            .method("GET")
            .willRespondWith()
            .headers(headers)
            .status(200)
            .body(responseBody).toFragment();
}
```
---

```json
{
  "provider" : {
    "name" : "FortuneService"
  },
  "consumer" : {
    "name" : "FortuneUi"
  },
  "interactions" : [ {
    "providerState" : "FortuneState",
    "description" : "a request for a random fortune",
    "request" : {
      "method" : "GET",
      "path" : "/random"
    },
    "response" : {
      "status" : 200,
      "headers" : {
        "Content-Type" : "application/json;charset=UTF-8"
      },
      "body" : {
        "id" : 8440038105,
        "text" : "JaizxIwBpBfLDObNbYRu"
      },
      "responseMatchingRules" : {
        "$.body.id" : {
          "match" : "type"
        },
        "$.body.text" : {
          "match" : "type"
        }
      }
    }
  } ],
  "metadata" : {
    "pact-specification" : {
      "version" : "2.0.0"
    },
    "pact-jvm" : {
      "version" : "2.1.13"
    }
  }
}
```

---

# Producer Pact Verification

```xml
<plugin>
	<groupId>au.com.dius</groupId>
	<artifactId>pact-jvm-provider-maven_2.11</artifactId>
	<version>2.1.13</version>
	<configuration>
		<serviceProviders>
			<serviceProvider>
				<name>FortuneService</name>
				<consumers>
					<consumer>
						<name>FortuneUi</name>
						<pactFile>../fortune-teller-ui/target/pacts/FortuneUi-FortuneService.json</pactFile>
					</consumer>
				</consumers>
			</serviceProvider>
		</serviceProviders>
	</configuration>
</plugin>
```

---

# [fit] PACT
# [fit] _Broker_
https://github.com/bethesque/pact_broker

---

![original fit](Common/images/pact_broker_4.png)

---

![original fit](Common/images/pact_broker_5.png)

---

![original fit](Common/images/pact_broker_6.png)

---

![270%](Common/images/dos_equis.jpg)
## Testing in _Production_

---

# [fit] _Blue_
# [fit] _Green_
# [fit] Deploys

---

![original fit](Common/images/blue_green.png)

---

# [fit] _Canary_
# [fit] Releases

---

![original fit](Common/images/canary_release.png)

---

# [fit] _MTBF_
# [fit] vs.
# [fit] _MTTR_

^ Mean Time Between Failures

^ Mean Time To Recovery

---

>Monitor the snot out of production and staging!
-- Elisabeth Hendrickson

---

# [fit] TO THE
# [fit] LABS!
