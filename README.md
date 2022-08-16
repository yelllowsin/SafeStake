# SafeStake

SafeStake is a decentralized validation framework for performing ETH2 duties and its backend is designed on top of [Lighthouse](https://github.com/sigp/lighthouse) (ETH2 consensus client) and [Hotstuff](https://github.com/asonnino/hotstuff) (a BFT consensus library).

## Architecture

Below is the architecture of SafeStake.

![alt](https://github.com/ParaState/SafeStake/blob/main/architecture.png?raw=true)

The SafeStake eco-system consists of several important stakeholders: SafeStake Service Provider, Validator, Operator.

### Validator

In ETH2, anyone can deposit 32 ETH to become a validator, in order to support ETH2's Proof of Stake consensus protocol. A validator is responsible for performing assigned duties and will get rewards if its work is submitted and acknowledged in time. Fail to actively participate in the duties result in penalties (gradually deduction of the initial 32 ETH balance). You can continue to the following links to learn how to become a validator for different nets:

- [mainnet](https://launchpad.ethereum.org/en/overview)

- [ropsten](https://ropsten.launchpad.ethereum.org/en/overview)

- [prater](https://prater.launchpad.ethereum.org/en/overview)

Based on the above introduction, it is critical that a validator should guarantee its availability online for timely responses to assigned duties. Moreover, *security* is another critical concern: inconsistent, dishonest, or malicious behavior will result in a more serious penalty, called ***slashing*** ([more on slashing](https://launchpad.ethereum.org/en/faq)). Therefore, there are two important requirements for maintaining a validator:

- High Availability

- Strong security (keep the validation key safe and avoid participating in slashable events)

**This is exactly what SafeStake provides under the hood.** It proposes a committee of operators (below) to collaborate in running the duties for a validator. Even if some operators are offline, others can still complete the tasks, which achieves high availability. Moreover, the private key is split among the operators, hence even if some of them are malicious or compromised, the private key is still safe and other honest operators can complete the tasks without being slashed, which achieves strong security.

### Operator

Briefly speaking, an operator is a party who holds a share of a validator's private validation key and signs duties with this key share. SafeStake uses a $(t,n)$-threshold BLS signature scheme to enable this feature. Namely, a validation key is split into $n$ shares, each of which is held by an operator. The key can NOT be reconstructed with less than $t$ shares. In the work flow, an operator can produce a signature share by signing a duty. Afterwards, if $t$ or more signature shares are collected, we can produce a valid signature that is equivalent to one signed by the original validation key. 

Before signing a duty, the committee of operators for a validator need to first agree on the duty to be signed. This requires a consensus protocol. Please be aware that *this consensus is NOT the ETH2 Proof of Stake consensus*. A BFT consensus protocol is enough for this purpose. SafeStake uses [***Hotstuff***](https://github.com/asonnino/hotstuff) to achieve the duty agreement among the committee of operators.

### SafeStake Service Provider

SafeStake provides services to enable the above features and connects validators to operators. In the core of its system, SafeStake provides a web service where:

- a user can register as an operator and join our operator pool

- a user who is a valid validator (has deposited 32 ETH beforehand) can choose a set of $n$ operators to run its duties.

These two points are detailed in the above architecture (i.e., user X is a validator, user Y is an operator).

## Get Started

In our eco-system, validators are delegating their tasks to operators and there is no need for deployment of validators. Therefore, we will discuss below two relevant deployment sections, one for *SafeStake Service Provider*, and one for *Operator*. Please only read the corresponding section for your deployment.

### Depoly SafeStake Service Provider

SafeStake service provider contains several components:

- A web server and frontend

- A nodejs backend (for necessary communication with operators)

- A root node service (for peer discovery in a p2p network)

#### Root Node Service

The duty agreement (using Hotstuff) among operators requires that these operators know IP of each other in a p2p network. Therefore, SafeSake provides a root node such that operators can consult and join the p2p network.

#### Dependencies
 * Public Static Network IP 
 * Hardware(recommend)
   * CPU: 4
   * Memory: 8G
   * Disk: 200GB
 * OS
   * Unix
 * Software
   * Docker
   * Docker Compose 


##### Installation

Clone this repository:

```shell
git clone --recurse-submodules https://github.com/ParaState/SafeStakeOperator dvf
cd dvf
```

Install Docker and Docker Compose

* [install docker engine](https://docs.docker.com/engine/install/)
* [install docker compose](https://docs.docker.com/compose/install/)

Build root node:

```shell
sudo docker compose -f docker-compose-boot.yml build
```

##### Start Service

Run the following to start the root node service:

```shell
sudo docker compose -f docker-compose-boot.yml up -d
```
Get root node enr

```
docker-compose -f docker-compose-boot.yml logs -f dvf_root_node | grep enr
```
output
> dvf-dvf_root_node-1  | Base64 ENR: *enr:-IS4QNa-kpJM1eWfueeEnY2iXlLAL0QY2gAWAhmsb4c8VmrSK9J7N5dfXS_DgSASCDrUTHMqMUlP4OXSYEVh-Z7zFHkBgmlkgnY0gmlwhAMBnbWJc2VjcDI1NmsxoQPKY0yuDUmstAHYpMa2_oxVtw0RW_QAdpzBQA8yWM0xOIN1ZHCCIy0*

NOTE: ***SafeStake should maintain such ENR(s) of root node(s) on its website, so that users who are registering as operators can use them to start operator nodes.***


### Depoly Operator
[https://github.com/ParaState/SafeStakeOperator](https://github.com/ParaState/SafeStakeOperator)

## Security Warnings

As of now, this project serves mainly proof-of-concepts, benchmarking and evaluation purpose and not for production use. Also implementation have not been fully-reviewed.
