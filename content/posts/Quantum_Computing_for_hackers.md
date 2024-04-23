---
title: "Quantum Computing for CTF Hackers"
date: 2024-04-21T21:42:37-07:00
draft: true
---
## Quantum Computing for CTF Hackers
Alright. So for the purposes of our discussion,
you can think of qubits as existing in some
account in Quantum Web Services (QWS) which
exposes the following API[^1].

## QWS Official API Documentation

```
func CreateSystem_v1 (number_of_qubits : int) -> SystemJSON
func CreateOperator_v1 (matrix : Matrix, system : SystemID) -> OperatorID
func Manipulate_v1 (system : SystemID, operator: OperatorID)
func Measure_v1 (qubit : QubitID) -> Pair<QubitID, bool>
```

* `CreateSystem_v1` intializes a specified number of new qubits. Behind the scenes
a classical system implementing this interface will implement a complex
vector of dimension $2^n$ that keeps track of the full state of the system. When we make calls to
`Manipulate_v1` and `Measure_v1` this state gets updated.

The SystemJSON response contains
```
{
    "SystemID" : SystemID
    "qubitIDs" : array<QubitID>
}
```

Where `SystemID` and `QubitID` are returned as strings. These are opaque
identifiers.

QWS will raise a `QuotaExceededException` if your account exceeds it's quota
of Qubits. Please contact QWS for support to raise this quota and discuss
billing details.

* `CreateOperator_v1` this is used to register operators on QWS. These
are how you modify the remote state of your system. The input is a
$2^n$ x $2^n$ matrix of complex values. It is required that the matrix's
inverse be equal to it's transpose (with the signs of any imaginary parts
of numbers swapped). Physics nerds would summarize this as "It's inverse
is equal to it's Hermitian (or Conjugate) Tranpose.

If the matrix does not satisfy these constraint, an `InvalidOperator` exception will
be raised from QWS and no OperatorID will be created.

* `Manipulate_v1` This will make a request to apply a previously specified operator
to the remote state. 

API Calls will return a NoSuchOperator Exception if the `OperatorID` is not associated with
the supplied system.

* `Measure_V1` When a qubit is measured

In addition to the above operators QWS provides the following convenience apis for
making it easier to create Operators (and save you bandwidth)

```
func CreateHadamard(qubit : QubitID) -> OperatorID
func CreateControlledNOT(qubit1 : QubitID, qubit2 : QubitID) -> OperatorID
```

# TODO: Actually talk a bit about how gates update the state

FAQ

1. Will QWS evntually provide the ability to create NonUnitary Operators?

No. QWS prides itself on maintaining the integrity of customer data. Non Unitary Operators
would irrevocably destroy customer data.

2. Will QWS provide nonlinear operators in future?

No. In addition to the above risk of losing customer data, this would violate causality and
potentially create a rip in the spacetime continuum. QWS cares about the Environment and this
is thus against the Company's 'Greener World' Initiatives.

3. Will QWS provide the ability to add Qubits to the system in future?

Yes, we are actively working on v2 of the API. In the further future, we will allow you
to also reference Qubits in systems other then ones you have created in your account.

When we do, upgrading the operators from being $2^n$ x $2^n$ matrices to $2^(n+1)$ x $2^(n+1)$
will be handled transparently.

[^1] If you prefer, a quantum system is a class in an OOP programming
language and things like the internal state of the system
are private variables. But this is a summary for hackers
who likely don't believe in taking things like `private` seriously.
It's memory on their computer dammit, they will damn well look at it
if they want to.

