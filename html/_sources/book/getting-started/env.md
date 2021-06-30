Flatland Environment
====================

The goal in Flatland is simple:

> **We seek to minimize the time it takes to bring all the agents to their respective target.** 

This raises a number of questions:

- [**Environment**](#Environment) where are the agents?
- [**Actions:**](#actions) what can the agents do?
- [**Observations:**](#observations) what can each agent "see"?
- [**Rewards:**](#rewards) what is the metric used to evaluate the agents?


🗺️ Environment
---

Flatland is a 2D rectangular grid environment of arbitrary width and height, where the most primitive unit is a cell. Each cell has the capacity to hold a single agent (train).

An agent in a cell can have a discrete orientation direction which represents the cardinal direction the agent is pointing to. An agent can move to a subset of adjacent cells. The subset of adjacent cells that an agent is allowed to transition to is defined by a 4-bit transition map representing possible transitions in 4 different directions.

![Imgur](https://i.imgur.com/Q72tAI8.png)
*8 unique cells enable us to implement any realworld railway network in the flatland env*

Agents can only travel in the direction they are currently facing. Hence, the permitted transitions for any given agent depend both on its position and on its direction. Transition maps define the railway network in the flatland world. One can implement any real world railway network within the Flatland environment by manupulating the transition maps of cells.
 
For more information on transtion maps checkout [level generation](../env/environment_information)!


↔️ Actions
---

The trains in Flatland have strongly limited movements, as you would expect from a railway simulation. This means that only a few actions are valid in most cases. 

Here are the possible actions:

- **`DO_NOTHING`**:  If the agent is already moving, it continues moving. If it is stopped, it stays stopped. Special case: if the agent is at a dead-end, this action will result in the train turning around.
- **`MOVE_LEFT`**: This action is only valid at cells where the agent can change direction towards the left. If chosen, the left transition and a rotation of the agent orientation to the left is executed. If the agent is stopped, this action will cause it to start moving in any cell where forward or left is allowed!
- **`MOVE_FORWARD`**: The agent will move forward. This action will start the agent when stopped. At switches, this will chose the forward direction.
- **`MOVE_RIGHT`**: The same as deviate left but for right turns.
- **`STOP_MOVING`**: This action causes the agent to stop.

Flatland is a discrete time simulation, i.e. it performs all actions with constant time step. A single simulation step synchronously moves the time forward by a constant increment, thus enacting exactly one action per agent per timestep.
```{admonition} Code reference
The actions are defined in [flatland.envs.rail_env.RailEnvActions](https://gitlab.aicrowd.com/flatland/flatland/blob/master/flatland/envs/rail_env.py#L69).

You can refer to the directions in your code using eg `RailEnvActions.MOVE_FORWARD`, `RailEnvActions.MOVE_RIGHT`...
```
### 💥 Agent Malfunctions
Malfunctions are implemented to simulate delays by stopping agents at random times for random durations. Train that malfunction can’t move for a random, but known, number of steps. They of course block the trains following them 😬.

👀 Observations
---

In Flatland, you have full control over the observations that your agents will work with. Three observations are provided as starting point. However, you are encouraged to implement your own.

The three provided observations are:
- Global grid observation
- Local grid observation
- Tree observation

![stock observations](https://i.imgur.com/oo8EIYv.png)

***Global, local and tree:** A visual summary of the three provided observations.*

**[🔗 Provided observations](env/observations)**

```{admonition} Code reference
The provided observations are defined in [envs/observations.py](https://gitlab.aicrowd.com/flatland/flatland/blob/master/flatland/envs/observations.py)
```

Each of the provided observation has its strengths and weaknesses. However, it is unlikely that you will be able to solve the problem by using any single one of them directly. Instead you will need to design your own observation, which can be a combination of the existing ones or which could be radically different.

**[🔗 Create your own observations](../env/custom_observations)**


🌟 Rewards
----------

At each time step, each agent receives a combined reward which consists of a local and a global reward signal. 

Locally, the agent receives $r_l = −1$ for each time step, and $r_l = 0$ for each time step after it has reached its target location. The global reward signal $r_g = 0$ only returns a non-zero value when all agents have reached their targets, in which case it is worth $r_g = 1$. 

Every agent $i$ receives a reward:

$$r_i(t) = \alpha r_l(t) + \beta r_g(t)$$

$\alpha$ and β are factors for tuning collaborative behavior. This reward creates an objective of finishing the episode as quickly as possible in a collaborative way.

In the [NeurIPS 2020 challenge](https://www.aicrowd.com/challenges/neurips-2020-flatland-challenge#background), the values used are: $\alpha = 1.0$ and $\beta = 1.0$.

```{admonition} Code reference
The reward is calculated in [envs/rail_env.py](https://gitlab.aicrowd.com/flatland/flatland/blob/master/flatland/envs/rail_env.py)
```

The episodes finish when all the trains have reached their target, or when the maximum number of time steps is reached. 

🚉 Other concepts
-----------------

### Custom levels

Going further, you will want to run experiment using a variety of environments. You can create custom levels either using multiple random generators, or design them by hands.

**[🔗 Generate custom levels](../env/level_generation)**

### Stochasticity

An important aspect of these levels will be their **stochasticity**, which means how often and for how long trains will malfunction. Malfunctions force the agents the reconsider their plans which can be costly. 

**[🔗 Adjust stochasticity](../env/stochasticity)**

### Speed profiles

Finally, trains in real railway networks don't all move at the same speed. A freight train will for example be slower than a passenger train. This is an important consideration, as you want to avoid scheduling a fast train behind a slow train!

```{note}
Speed profiles are not used in the first round of the [NeurIPS 2020 challenge](https://www.aicrowd.com/challenges/neurips-2020-flatland-challenge/).
```

**[🔗 Tune speed profiles](../env/speed_profiles)**