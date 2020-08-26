Public mini web-site https://jips-org.github.io/multi-mno-flowkit/

# Introduction

In 2018 the Joint IDP Profiling Service embarked on a journey exploring frameworks and techniques for data anonymisation beginning with the use and application of the Statistical Disclosure Control tool SDCmicro. In March 2019 JIPS was awarded a grant by the UNHCR Innovation Fund to deepen the research on advanced data science methods for data anonymisation. We were able to discuss state of the art methods such as Multi Party Computation and Homomorphic Encryption through the exchange with technical experts such as the Johns Hopkins University Applied Physics Lab and Flowminder as well as experts among the data producers and users such as the Victims Unit of the Government of Colombia’s National Statistics Office. This phase of the project concluded in December 2019 with an in-depth desk review outlining methods for the development of a prototype workflow. The goal of the prototype was to enable actors in the humanitarian and development field working with microdata to safely access and query sensitive individual-level data without needing to physically transfer it (share it). 
In January 2020 JIPS was awarded a follow up grant by the UNHCR Innovation Fund to develop the prototype and document the process. In this phase we consulted with a variety of stakeholders including UNHCR Regional Offices, Joint Projects and other stakeholders on the programmatic side in order to develop the prototype around a concrete data source. As this proved challenging, the project had to shift to making use of simulated Call Detail Records (CDR) in close collaboration with Flowminder and using the already tried and tested approaches used in the Flowkit. 
The main questions that we set out to answer through this project were:

 * how can we safely query microdata behind the owner's firewall?
 * how can we make sure the query itself is anonymised?
 * how can we safely export the results of the query?
    
Given the above limitations, the goal of this phase of the project was to demonstrate the safe use of CDR data coming from several Mobile Network Operators (MNOs) while respecting the highest data privacy standards and the "General Data Protection Regulation" (GDPR).
GSMA recommends in their [guideline](https://www.gsma.com/publicpolicy/consumer-affairs/privacy) to never export RAW or even pre-anonymised data from the MNO's infrastructure. Only anonymised and aggregated data can be exported.

## JIPS/ FlowMinder / ToggleCorp / Alsenet

For this project [JIPS](http://jips.org) partnered with FlowMinder, ToggleCorp and Alsenet.

* FlowMinder provides CDR analysis package under Free Software license
* ToggleCorp provides data analysis and has been involved as a technical implementation partner in this project since its onset
* Alsenet provides technical project management and IT infrastructure for the project

## Simulation

For the purpose of this project we created a demo dataset simulating 3 months of data from 3 MNOs in Nepal. The data is simulating the Nepal earthquake of 2016. The goal is to track flows of population above normal to more efficiently plan rescue operations.

After the Earthquake responding partners have contacted several (3) MNOs and are interested in understanding aggregated numbers of people moving from destination A to B at an above normal flow. Therefore, individual level data needs to be accessed on premises of the MNOs who will provide an anonymised subset of this data to the requesting partner. 

## Personally Identifiable Information (PII) - ideal VS reality

In the ideal setup the NGO  provides Free Software and a corresponding documentation to the MNO extract aggregated and anonymised data. In this ideal setup only the MNO has access to RAW data, to de-identified data and  has to agree with the NGO on the type of queries that can be accessed via the API.
The following requirements are extracted from the [GSMA guidelines on protecting privacy in the use of mobile phone data for responding to the Ebola outbrake](https://www.gsma.com/mobilefordevelopment/wp-content/uploads/2014/11/GSMA-Guidelines-on-protecting-privacy-in-the-use-of-mobile-phone-data-for-responding-to-the-Ebola-outbreak-_October-2014.pdf)

1. The mobile phone numbers of subscribers making and receiving calls or text messages will be anonymised by mobile operators inside their premises and on their equipment. This is achieved by replacing the mobile phone numbers with an anonymous code before being analysed. This is done through a hashing process using the secure SHA-3 algorithm.
2. Anonymised CDR data will not be transferred outside of the operator’s system/premises: The anonymised data will be kept secure and encrypted within the operator’s premises. Access to the data will be controlled and given only to pre-approved and authorised personnel. A record of access will be maintained and auditable. Access to the algorithm and the ability to decrypt the data will be further protected by also limiting access to pre-approved and authorised personnel.
3. All analysis will take place onmobile operator’s systems, in their premises and under operator supervision.Once anonymised by the mobile operator, the data will be analysed by approved research entities that agree to abide by strict ethical standards on the use of data.
4. No analysis will be undertaken that singles out identifiable individuals.No attempts will be made to link the data to other data about an individual and which may impact on their privacy or otherwise cause harm.
5.Only the output of the analysis (i.e. the resulting non-sensitive data on population mobility estimates, aggregate statistics, indicators, etc.)will be made available to relevant and approved aid agencies,government or research agencies that can use these inputs in their modelling and planning efforts.No sensitive data will be shared with or made available to any third parties. 

In reality, some MNOs do not have the necessary infrastructure and IT ressources to deploy all the necessary software stack themselves. In this case a trusted party such as FlowMinder or ToggleCorp can deploy the stack and process the data.

# Deployment

## Proposed architecture 

The NGO will operate a public webserver to present the aggregated results. The NGO will also run a server or a workstation for using the FlowKit Client to access FlowKit APIs made available by the different MNOs via an encrypted channel (VPN).

![](https://github.com/jips-org/adsm/blob/master/documentation/Multiple-CDR-providers.png?raw=true)

## Hardware requirements


### Servers per MNO

Analysing CDR data requires a lot of resources. We suggest a powerful server with at least :

* 16+ CPU cores
* 50T+ storage
* 128G+ RAM memory

### Public web server

Depending on the needs of the NGO the requirements for the public web server are ranging from a static website hosted on IPFS or github pages to a small VM with database and server-side backend. In our case github pages were used to host and to present the static content resulting from aggregated and anonymised data.

## Network infrastructure 

### DMZ

Usually FlowKit server or any software external to the MNO will be hosted inside a DMZ (demilitarized zone, sometimes referred to as a perimeter network or screened subnet). The DMZ has no direct access to the MNO's network and  has a restricted Internet connectivity, usually accessible only via a VPN.

### Firewall & VPN

The MNO provides a firewall allowing a VPN connection from the “FlowKit client” workstation to FlowKit servers hosted in the MNO's DMZ. 

## Installing FlowKit

The FlowKit software deployment is discribed here. Following this documentation the MNO can deploy the software, import its pseudo-anonymised data, review and validate queries that can be accessed via the FlowKit API.

## Raw data de-identification

A [pseudo-anonymisation script](https://github.com/jips-org/adsm/blob/master/data_generation/pre_anonymize.py) is executed on RAW data by the MNO and the resulting data is examined by the MNO before releasing it to be imported into the FlowKit software being installed on MNO's infrastructure.

RAW data contains PII such as MSISDN (phone number) and depending on the MNO may have additional PII.
```
datetime,msisdn,location_id
2016-01-01 00:00:00.055125+00:00,9779703021535,ab2a016fdeac0f1b2a6cea2050840f41
2016-01-01 00:00:02.007651+00:00,9779796988097,63b0ef0f04e7415a42cb5c919b1455fc
```

De-identified data only contains salted and pepered hash instead of PII
```
datetime,msisdn,location_id
2016-01-01 00:00:00.055125+00:00,88f08fd8d5941ced11a3ae6499ee38f7837847f776f270485f2d6746c4cb84d7,ab2a016fdeac0f1b2a6cea2050840f41
2016-01-01 00:00:02.007651+00:00,6c6d84d365344461a58f17087fe8fdfc5a48b9805c587c57d2a7228edf98afd6,63b0ef0f04e7415a42cb5c919b1455fc
```

We used the SHA-256 hashing algorithm with added salt and peper to prevent phone number bruteforcing (or any other PII)

A static pepper value is used to protect against dictionary attacks, the pepper value is added to all input values before hashing. If you intend to use the hash as join or lookup keys, then the pepper should be the same for all pseudonymized datasets.

Optionally a salt column can be added to protect against dictionary attacks, the value in each row of this column will be added to the corresponding row of the input values before hashing. If you intend to use the hash as join or lookup keys, then the salt column should be present and identical for all pseudonymized datasets.

𝐻𝐴𝑆𝐻(𝑟𝑜𝑤𝑖+𝑝𝑒𝑝𝑝𝑒𝑟+𝑠𝑎𝑙𝑡𝑖)

Additionally, temporal and spatial resolution can be reduced by the MNO in close collaboration with the NGO to be sure that NGO needs are matched.

You can learn more about spatial and temporal resolutions of the CDR aggregates on https://covid19.flowminder.org/cdr-aggregates/understanding-cdr-aggregates-fundamentals

## Importing the data into FlowKit

Pre-anonymised and de-identified CSV files need to be copied by a secure channel (such as scp or rsync over ssh) to `/data/pre_anonymized_data/telecom_name`

New files can be imported on daily basis and the FlowKit software will automatically process them and will make available the aggregated information via API pre-approved queries.

## Auditing and authorising API queries

Only audited queries are added and authorised to be executed on the pre-anonymised datasets. The MNO remains in full control of the RAW and pseudonomous data. The MNOs also audits and authorises specific API queries. Some worked examples are available on https://flowkit.xyz/analyst/worked_examples/

For this demo we used the [Flows Above Normal](https://flowkit.xyz/analyst/worked_examples/flows-above-normal/). 

As you can see the [Flows Above Normal](https://flowkit.xyz/analyst/worked_examples/flows-above-normal/) example aggregates people flows to the administrative level 3 that we considered to provide sufficient group anonymity for the needs of this demo.

`aggregation_unit="admin3"   # Use administrative level 3 regions as the locations`

# Queries

The Jupyter notebooks we used is documented [here](https://github.com/Flowminder/FlowKit/blob/master/docs/source/analyst/worked_examples/flows-above-normal.ipynb).

We adapted the Jupyter notebook to use multiple MNOs and to combine the aggregated data of the MNOs together https://github.com/jips-org/adsm/blob/master/outputs/jips_multisource_displacement.ipynb

# Results 

The results of running FlowKit client against the 3 API servers exposed by the MNOs are presented [here](https://jips-org.github.io/multi-mno-flowkit/)

As you can see, we successfully combined aggregated and pseudonymized data from 3 MNO while respecting the GSMA guidelines on data privacy.

# Conclusions

The project achieved the following:
- The goal of this phase of the project was to demonstrate that it is possible to perform data analysis within the data owner/provider infrastructure and only export anonymised and aggregated data. We developed a technical workflow where we were able to demonstrate this with one single data provider, we achieved a good understanding of the problems and limitations in case of multiple data providers and explored some possible paths to adapting the workflow also for those situations.
- We have set up a microsite documenting the process and giving access to the code which makes our prototype and its simulation reproducible
- We have generated simulation data for the purpose of this exercise – which is valuable initself and can be used for the purpose of refinement and exploration of the methods
- We explored two state-of the art advanced data science methods as planned: Multi-Party Computation and Homomorphic Encryption and have documented those evaluations
- We engaged in a number of discussions and presentations to key UNHCR stakeholders including Senior Management and the DHC/HC exploring the use and application of these techniques for CASH Based Interventions data through the Joint Targeting Project and others
- We have captured the evaluation of the cryptographic methods and main lessons learned in an additional article which can be discussed with the wider community and feed into related initiatives

## Limitation of this approach

In this example we combined already aggregated and de-identified data on the "admininistrative region level 3" - that was considered as non PII. This would not be possible to combine data on any of the PII fields. For example a COVID-19 analysis would imply to combine PII from different MNO. Proper usage of spiced hashes makes this impossible.

## Other methods

Other methods we investigated includes:

* MPC
* PPRL/ALC
* FHE
* ZKP & ABC

### Privacy Preserving Record Linkage / Anonymous Linking Codes (ALC)

If the PII fields like **Firstname Lastname** are manually collected and may include typos or misspellings - it will be impossible to match them. This is where CLK Hash is helpgul. It's an error tollerant hashing algorithm that provides a probabilistic matching even with a few minor errors.

We evaluated a particular implementation called [CLK Hash](https://clkhash.readthedocs.io/en/latest/) that implements anonymous linkage service with [Anonlink](https://github.com/data61/anonlink)

Anonlink computes similarity scors, and/or best guess matches between sets of cryptographic linkage keys (hashed entrey records).

Clkhash is used to create a cryptographic linkage key from personally identifiable data (PII).

A Python (and optimised C++) implementation of anonymous linkage using cryptographic linkage keys as described by Rainer Schnell, Tobias Bachteler, and Jörg Reiher in A Novel Error-Tolerant Anonymous Linking Code.

###  Zero Knowledge Proofs (ZKP) & Attribute Based Credentials (ABC)

In cryptography, a zero-knowledge proof is a method by which one party (the prover) can prove to another party (the verifier) that they know a value x, without conveying any information apart from the fact that they know the value x. The essence of zero-knowledge proofs is that it is trivial to prove that one possesses knowledge of certain information by simply revealing it; the challenge is to prove such possession without revealing the information itself or any additional information. 

Attribute-based credentials are cryptographic schemes designed to enhance user privacy. These schemes can be used for constructing anonymous proofs of the ownership of personal attributes. The attributes can represent any information about a user, e.g., age, citizenship or birthplace. The ownership of these attributes can be anonymously proven to verifiers without leaking any other information. 

https://zenroom.org/ people from https://www.dyne.org/ were very kind and provided us a dedicated workshop on how to use ABC with Zenroom.

https://dev.zenroom.org/#/pages/zencode?id=attribute-based-credentials

ZKP & ABC are the building blocks of the digital self-sovereign identity. Zenroom is part of the [DECODE project](https://decodeproject.eu/) about data-ownership and technological sovereignty. The mission of the project is to improve people's awareness of how their data is processed by algorithms, as well facilitate the work of developers to create along privacy by design principles using algorithms that can be deployed in any situation without any change.
