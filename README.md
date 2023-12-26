# **Abstract**

During the research phase of our solution-building, we were inspired by the division of the operational plan of the airlines into two main dimensions: flights and passengers. Our general approach is to identify the affected passengers, shortlist the potential alternate flights and develop a passenger score and a flight score. From these two scores, we were able to build an objective function to optimize and guide the respective passengers to the best possible flight alternatives using QUBO (Quadratic Unconstrained Binary Optimization).

# **Classical Approach**

Before settling on the quantum optimization technique mentioned earlier, we tried a classical approach to solve the given problem. This helped us better understand the problem statement as well as the need for a quantum approach. After much discussion, we decided on a three-step procedure as follows:

## 1. **PNR Ranking** :

According to the given Rules Set, we define a function to get the priority of affected passengers. Then, we arrange these passengers from the highest priority to the least priority.

## 2. **Route Planning** :

We had information about cancelled flights, which we represented as an array. To adapt to these disruptions, we modified the graph by removing the edges corresponding to the cancelled flights. This created a new graph that reflected the available flight options after the cancellations.

That's where we used Dijkstra's Algorithm. We started with the origin city as the initial node, assigned tentative travel times to all other nodes (set initially to infinity), and set the tentative travel time to the origin city as zero.

For each neighbouring node, we updated its tentative travel time based on the sum of the current tentative travel time and the weight of the connecting edge. We chose the node with the smallest tentative travel time as the next node to explore, repeating this process until we reached the destination city.

In addition to flight durations, we added an attribute to the edges representing the time between connecting flights. This attribute accounted for the time we needed to wait at a connecting city before catching the next flight.

As we traversed the graph using Dijkstra's Algorithm, we kept track of the total travel time, including both flight durations and connecting times.

The result of our process was a path from the origin city to the destination city that represented the best suitable flight alternatives, considering the disruptions. This approach allowed us to effectively navigate through disruptions, find alternative flight options, and minimize the overall travel time, providing a more efficient and reliable travel plan.

## 3. **Re accommodation** :

After the route planning, we start accommodating the affected passengers to their alternative flights based on the priority assigned to them. If the number of passengers affected is greater than the available seats in the best alternate flight, then we accommodate the rest in the next best alternative. This process continues until all the affected passengers are accommodated.

# **Quantum Approach**

Our objective is to maximize the number of passengers to whom we can provide alternate flights. We have scores for each flight and each affected passenger. The flight scores are used to find alternate flights. After finding alternate flights our job is to accommodate the passengers in those flights. We construct an objective function for this.

![obj_Function](https://github.com/Standby-Coder/qc-flight-rescheduler/assets/152943694/c2b75aa8-01dc-4272-bdc8-1674e9cc14f7)


We can construct our problem as a QUBO (Quadratic Unconstrained Binary Optimization) problem by converting the constraints to penalty terms that can be added to the objective function. Our variables are, i.e. a mapping of each passenger to each alternate flight.

The number of variables will be the number of passengers times the number of alternate flights, which is quite large. So, it would be very difficult to solve this kind of problem classically.

Here, we can utilise the computing power of quantum computers.

However, even in quantum computing optimization, the number of qubits needed to represent the problem can limit current quantum hardware as mentioned in the paper "How many qubits are needed for quantum computational supremacy?" by [Alexander M. Dalzell](https://arxiv.org/search/quant-ph?searchtype=author&query=Dalzell,+A+M), [Aram W. Harrow](https://arxiv.org/search/quant-ph?searchtype=author&query=Harrow,+A+W), [Dax Enshan Koh](https://arxiv.org/search/quant-ph?searchtype=author&query=Koh,+D+E) and [Rolando L. La Placa](https://arxiv.org/search/quant-ph?searchtype=author&query=La+Placa,+R+L). We explored different methods to solve combinatorial problems and delved into QAOA.

## **Quantum Approximate Optimization Algorithm (QAOA)**

QAOA is a quantum optimization procedure to solve combinatorial problems. It is a gate-based algorithm i.e. it employs quantum gates to manipulate quantum states and gradually adjusts parameters to find the solution. It gives approximate solutions. The current state of quantum hardware imposes limitations on the size of problems that can be solved practically.

While QAOA has limitations, especially on near-term quantum devices due to noise, it can still be advantageous for solving large problems compared to classical methods.

## **Implementation**

We tried to solve our problem using the Qiskit Optimization framework using the QAOA algorithm and COBYLA optimizer. QAOA can't handle QUBO problems directly. They have to be converted into Ising Hamiltonian, which can be done using a QAOA solver in Qiskit.

We tried two test cases:

(1) Cancelling one flight and rescheduling its passengers. Based on which flight was cancelled, flight scores were calculated and the best alternate flights (2 in our case) were obtained. The affected passengers(255 in total) were identified based on the cancelled flight. Scores were calculated for each affected passenger.

(2) Aircraft Type change which affected the capacity of the flight. The number of passengers who couldn't be accommodated had to be identified and our optimization procedure was implemented. Our test case had 120 passengers and 2 alternate flights.

In both the cases the number of variables were quite large (255\*2 = 510 in (1) and 120\*2 = 240 in(2)). This many variables can't be handled by any quantum optimizer accessible to us. So we tried some optimization procedures.

## **Optimization**

Due to the hardware limitations of current quantum optimizers, we optimized our problem by reducing the number of variables. For this we attempted at grouping the passenger data based on their scores assigned i.e. we'll consider passengers having similar scores as a single entity or individual. This will reduce the number of variables drastically. We could group the passengers into a maximum of 5 groups for the solver to process the problem. These 5 groups will now be treated as individuals. We now have 5\*2 = 10 variables only. Our constraint equations will modify accordingly. Now the second constraint (flight capacity constraint) equation will also include a coefficient that represents the number of passengers in that group.

## **Results**

Our problem was converted to a minimization problem when it was converted into QUBO. QAOA was run and solutions were obtained. We analysed the first 4 optimal (having the maximum value of our objective function) solutions obtained.

![result_img](https://github.com/Standby-Coder/qc-flight-rescheduler/assets/152943694/c3ee17cf-7a6b-4591-8df4-d5f8606f8ac2)

**Figure 1: QAOA convergence in our problem**

**Table 1: Obtained Results of first four optimal solutions (One flight cancellation)**

| Number of passengers | Solution 1 | Solution 2 | Solution 3 | Solution 4 |
| --- | --- | --- | --- | --- |
| Accommodated | 255 | 255 | 205 | 205 |
| Assigned the same cabin | 94 | 73 | 44 | 57 |
| Received an upgrade | 96 | 106 | 97 | 103 |
| Received a downgrade | 65 | 76 | 64 | 45 |

**Table 2: Obtained Results of first four optimal solutions (Aircraft Type Change)**

| Number of passengers | Solution 1 | Solution 2 | Solution 3 | Solution 4 |
| --- | --- | --- | --- | --- |
| Accommodated | 125 | 125 | 125 | 125 |
| Assigned the same cabin | 9 | 11 | 24 | 9 |
| Received an upgrade | 90 | 93 | 84 | 92 |
| Received a downgrade | 26 | 21 | 17 | 24 |

# **Further work:**

## **Multiple flights**

We tested our algorithm on a single flight cancellation. It can also work for multiple flight cancellations. For this, we have to impose some additional constraints.

![additional_const](https://github.com/Standby-Coder/qc-flight-rescheduler/assets/152943694/c4200119-c444-4402-bb6c-d9b296b41cef)


## **Connecting flights**

If there are connecting flights available between the departure and destination airports of the affected flight we can consider them also. For example, if there are two flights one from A to B and the other from B to C, A being the starting point and C being the destination, we can consider them provided the available seats on the second flight are greater than or equal to that of the first otherwise the journey couldn't be completed for some passengers. Both the flights can then be considered as a single flight going from A to B. Similarly different combinations of connecting flights can be considered as one flight and our reaccommodation procedure can be followed. _(as explained in Table 2)_

![table2_img](https://github.com/Standby-Coder/qc-flight-rescheduler/assets/152943694/378e3b30-b415-48cf-855c-2f508a11cd26)

**Table 2: Connecting Flights Aggregation Examples**

# **OTHER EXPLORATORY METHODS:**

## **Quantum Annealing**

The optimisation method with QAOA and gates is very slow and prone to errors so we researched on other exploratory methods while optimising our objective function. One of which was using quantum annealers. In our optimization problem, we are looking for the best combination of a superposition of rescheduling the flight for maximum passengers with the minimum possible delay time with priority based on PNR scores. Quantum annealing simply uses quantum physics to find low-energy states of a problem and therefore the optimal or near-optimal combination of elements.

Here, we use qubits which can exist as a superposition of 0 and 1 states. However, at the end of the quantum annealing process, the qubit collapses to a classical bit (0 or 1). Here the presence of a passenger on a particular flight is represented by a qubit. The priority and the presence of the other passengers are brought about in the system by _Bias_ and _Coupling_.

### 1. **BIAS** 

With no biases, the probability of the qubit ending in the 0 or the 1 state is equal (50 percent). We control the probability of it falling into the 0 or the 1 state by applying an external magnetic field to the qubit. This field tilts the double-well potential, increasing the probability of the qubit ending up in the lower well. The programmable quantity that controls the external magnetic field is called a _bias_, and the qubit minimises its energy in the presence of the bias. We use this bias to give priority to the passenger with a higher PNR score with a better flight score.

### 2. **COUPLING** 

We use a coupler to induce the phenomenon of quantum physics called entanglement. When two qubits are entangled, they can be thought of as a single object with four possible states. The two qubits exist in 4 possible states together (0,0), (0,1), (1,1), and (1,0). The relative energy of each state depends on the biases of qubits and the coupling between them. During the anneal, the qubit states are potentially delocalised in this landscape before finally settling into a classical state at the end of the anneal. We use this phenomenon to couple all the passengers with each other in our problem. ![]

![annealing_img](https://github.com/Standby-Coder/qc-flight-rescheduler/assets/152943694/bc770cf8-b8ce-4669-b8a0-5b98bd95b329)

**Figure 2: Biasing of qubit over time during quantum annealing**

For a quantum system, a Hamiltonian is a function that maps certain states, called _eigenstates_, to energies. In quantum annealing, the system begins in the lowest-energy eigenstate of the initial Hamiltonian. As it anneals, it introduces the problem Hamiltonian, which contains the biases and couplers, and it reduces the influence of the initial Hamiltonian.

At the end of the anneal, it is in an eigenstate of the problem Hamiltonian. Ideally, it has stayed in the minimum energy state throughout the quantum annealing process so that—by the end—it is in the minimum energy state of the problem Hamiltonian and therefore has an answer to the problem we want to solve. By the end of the anneal, each qubit is a classical object.

Here the collection of the flights and the passengers on it and the various possibilities gives the various Hamiltonian states which are minimised through annealing while using couplers and biases, to give us the most optimal solution for the passengers to be rescheduled. As the number of passengers increases, the complexity increases and with each added qubit we double the number of scenarios. This gives us a wide spectrum of energies represented by the Hamiltonian to optimise.

The Variational Quantum Eigensolver (VQE) for passenger rescheduling involves formulating the scheduling problem into a quantum Hamiltonian, initializing a quantum state, and iteratively optimizing it through a parameterized quantum circuit. The expectation value of the Hamiltonian is minimized using classical optimization algorithms, guiding the adjustment of the quantum circuit's parameters. The optimized parameters yield a quantum state that represents an approximate solution to the ground state of the Hamiltonian. The final decoded quantum state provides an optimal or near-optimal solution to the passenger rescheduling problem, balancing the accommodation of passengers with minimal flight delays.

![global_minima_img](https://github.com/Standby-Coder/qc-flight-rescheduler/assets/152943694/603593c9-d449-4314-a6c4-ac49eb9182e4)

**Figure 3: Quantum Annealing used to find Global minima**

## **JuliQAOA**

The simulation of QAOA can be done in various environments such as Qiskit or Pennylane.

We simulated the QAOA on Qiskit. The computation time was around 10 minutes. So, we started looking for alternate faster implementations. JuliQAOA is one such package. In JuliQAOA, the cost and mixer Hamiltonians are pre-computed and supplied to the simulator along with angles, resulting in a substantial reduction in computation time.

Initially, for a given optimization problem characterised by a cost function 𝐶(𝑥), the user computes and stores the values of 𝐶(𝑥) across all feasible states S. The second pre-computed quantity is the mixer Hamiltonian, which can be calculated for both unconstrained and constrained problems.

JuliQAOA's approach of pre-computation and purpose-built simulation surpasses existing alternatives. Its requirement of only a list of 𝐶(𝑥) values across feasible states provides flexibility in choosing cost functions. Notably, JuliQAOA excels in constrained optimization, eliminating the need for artificial "penalty" terms in traditional circuit-based simulators. This unique design allows the use of mixers confined within the feasible subspace, disregarding non-feasible states. This enhances accuracy and significantly reduces computational efforts.

# **References: **

1. D Wave's Problem solving handbook [https://docs.dwavesys.com/docs/latest/doc\_handbook.html](https://docs.dwavesys.com/docs/latest/doc_handbook.html)

2. Topologies of QPU Architecture by D Wave [https://docs.dwavesys.com/docs/latest/c\_gs\_4.html#getting-started-topologies](https://docs.dwavesys.com/docs/latest/c_gs_4.html#getting-started-topologies)

3. Qiskit optimization overview

[https://qiskit.org/ecosystem/optimization](https://qiskit.org/ecosystem/optimization/)[/](https://qiskit.org/ecosystem/optimization/)

4. "_JuliQAOA: Fast, Flexible QAOA Simulation_" by

[John Golden](https://arxiv.org/search/quant-ph?searchtype=author&query=Golden,+J), [Andreas Bärtschi](https://arxiv.org/search/quant-ph?searchtype=author&query=B%C3%A4rtschi,+A), [Daniel O'Malley](https://arxiv.org/search/quant-ph?searchtype=author&query=O%27Malley,+D), [Elijah Pelofske](https://arxiv.org/search/quant-ph?searchtype=author&query=Pelofske,+E), [Stephan Eidenbenz](https://arxiv.org/search/quant-ph?searchtype=author&query=Eidenbenz,+S)

[https://arxiv.org/abs/2312.06451](https://arxiv.org/abs/2312.06451)

5. "_How many qubits are needed for quantum computational supremacy?_" by

[Alexander M. Dalzell](https://arxiv.org/search/quant-ph?searchtype=author&query=Dalzell,+A+M), [Aram W. Harrow](https://arxiv.org/search/quant-ph?searchtype=author&query=Harrow,+A+W), [Dax Enshan Koh](https://arxiv.org/search/quant-ph?searchtype=author&query=Koh,+D+E), [Rolando L. La Placa](https://arxiv.org/search/quant-ph?searchtype=author&query=La+Placa,+R+L)

https://arxiv.org/abs/1805.05224

