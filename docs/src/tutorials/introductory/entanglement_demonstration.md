# Entanglement demonstration tutorial

```@meta
DocTestSetup = quote
    ENV["ANYON_QUANTUM_USER"] = "test-user"
    ENV["ANYON_QUANTUM_TOKEN"] = "not-a-real-token"
    ENV["ANYON_QUANTUM_HOST"] = "yukon.anyonsys.com"
end
```

This tutorial is going to demonstrate entanglement by preparing and measuring a bell state.

## Theory

Bell states are a set of maximally entangled states. We are going to look at one of these states, $\frac{\left|00\right\rangle+\left|11\right\rangle}{\sqrt{2}}$, in this tutorial.

The $\frac{\left|00\right\rangle+\left|11\right\rangle}{\sqrt{2}}$ state can be constructed using the following circuit.

```@raw html
<div style="text-align: center;">
	<img
		src="../../images/entanglement_circuit.svg"
		title="Bell-state generator"
        style="transform: scale(2);margin: 2em"
	/>
</div>
```

The Hadamard gate on the first qubit puts the first qubit into state $\frac{\left|0\right\rangle+\left|1\right\rangle}{\sqrt{2}}$. Since the second qubit remains unchanged, the total system is in state $\frac{\left|00\right\rangle+\left|10\right\rangle}{\sqrt{2}}$ after applying the controlled-X (CX) gate on qubits one and two.

If the first qubit is in state zero then nothing happens to the second qubits. State $\left|00\right\rangle$, therefore, stays the same.

If the first qubit is in state one the second qubit is flipped. State $\left|10\right\rangle$, therefore, becomes state $\left|11\right\rangle$. Therefore, after the controlled-X gate the system's state is $\frac{\left|00\right\rangle+\left|11\right\rangle}{\sqrt{2}}$.

The two qubit's states are now entangled. If you measure one of the qubits you also get information about the other qubit's state. They are no longer independent.

## Code

We are going to start by importing Snowflake and creating our circuit.

```jldoctest entanglement_demonstration_tutorial; output = false
using Snowflake

circuit = QuantumCircuit(qubit_count = 2)

# output

Quantum Circuit Object:
   qubit_count: 2
q[1]:

q[2]:
```

We must now apply our gates to our circuit.

```jldoctest entanglement_demonstration_tutorial; output = false
push!(circuit, hadamard(1), control_x(1, 2))

# output

Quantum Circuit Object:
   qubit_count: 2
q[1]:──H────*──
            |
q[2]:───────X──
```

Now we want to run this tutorial on Anyon's Quantum computer. We need to construct an AnyonQPU object. You can get more information on QPU objects at the [Get QPU Metadata tutorial](./get_qpu_metadata.md).

```jldoctest entanglement_demonstration_tutorial; output = false
user = ENV["ANYON_QUANTUM_USER"]
token = ENV["ANYON_QUANTUM_TOKEN"]
host = ENV["ANYON_QUANTUM_HOST"]

qpu = AnyonQPU(host=host, user=user, access_token=token)

# output

Quantum Processing Unit:
   manufacturer:  Anyon Systems Inc.
   generation:    Yukon
   serial_number: ANYK202201
   qubit_count:   6
   connectivity_type:  linear
```

We cannot run our circuit directly on the QPU since neither the Hadamard gate nor the CX gate is a native gate of Anyon's Quantum Computer. We first have to transpile the circuit.

```jldoctest entanglement_demonstration_tutorial; output = false
transpiler = get_transpiler(qpu)
transpiled_circuit = transpile(transpiler, circuit)

# output

Quantum Circuit Object:
   qubit_count: 2 
Part 1 of 2
q[1]:──Z_90────────────X_90────Z_90────────────────────*──────────
                                                       |          
q[2]:──────────Z_90────────────────────X_90────Z_90────Z────Z_90──
                                                                  

Part 2 of 2
q[1]:────────────────
                     
q[2]:──X_90────Z_90──
```

Now we run our quantum circuit on Anyon's quantum computer!

```julia
num_repetitions = 200
result = run_job(qpu, transpiled_circuit, num_repetitions)

println(result)
```

The results show that the samples are mostly sampled between state $\left|00\right\rangle$ and $\left|11\right\rangle$.

```text
Dict("11" => 97, "00" => 83, "01" => 11, "10" => 9)
```

## Summary

In this tutorial, we've shown how to generate one of the Bell states. With this we've demonstrated entanglement between two qubits.

The full code for this tutorial is available at [tutorials/entanglement\_demonstration.jl](https://github.com/anyonlabs/Snowflake.jl/blob/main/tutorials/entanglement_demonstration.jl)