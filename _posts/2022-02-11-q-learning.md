---
layout: post
title:  "Solving Reinforcement Learning"
date:   2022-02-11
categories: Reinforcement Learning
---

In the post about reinforcement learning we looked at the key concepts of reinforcement learning and what problem we actually want to solve. Now we want to adress how to solve this problem.

Before we can combine the faszinating part of understanding how we can sove the problem we need to clarify one more concept. And this is state vaue and action value functions. Let's get started.
 

# State Value Function and Action Value Function
A state value function $$V: S \to \mathbb{R}$$ maps states to a value. This value represents the expected return the agent gets if he starts in a state and then acts accoring to its policy. 

$$V^{\pi}(s) =  \mathbb{E}_{\pi}[R_t(\tau) | S_t=s] = \mathbb{E}_{\pi} [\sum_{k=0}^{\infty} \gamma^k r_{t+1+k}]$$

How can this value help us? In the end we are interested in a policy $$\pi$$ and that is a mapping from state to action.

One approach is to evaluate for each possible action in state s what the reward belongs to this action and what is
the expected return of the state s' we end up by this action.

$$a = \operatorname*{arg\max}_{a \in A}\mathbb{E}_{\pi}[R_a(s,s') + V(s')]$$

This is a valuation solution in a MDP, as there we know the state transition function, meaning that we know in which 
state S' we end if we execute action a in state s. In reinforcement learning we unfortunately don't know this.

Thus, instead of storing only the expected return for a state we store the expected return for each action we can do in
that state. This function is called action value function.

$$Q^{\pi}(s,a)= \mathbb{E}_{\pi}[R_t(\tau) | S_t=s, A_t=a]$$

Having the q-values stored for each state we can easily chose the action that returns the highest expected return.

$$Q(s,a)=\operatorname*{arg\max}_{a \in A}Q(s,a)$$


## Optimal Value Function
We already talked a few times about the optimal value function and the improved or optimal policy. We intuitively understand what is meant by that. But we missed a formal definition so far. 
A policy $$\pi$$ is better than a policy $$\pi'$$:

$$\pi \geq \pi' \text { if and only if } V^{\pi}(s) \geq V^{\pi'}(s), \forall s \in S$$

The optimal value function is defined as:

$$V^*(s) =  \operatorname*{max}_{\pi} V^{\pi}(s)$$

Analogusly, we can define the optimal action value function:

$$Q^*(s,a) = \operatorname*{max}_{\pi} Q^{\pi}(s,a)$$

# Solving the Grid World
Finally, we are ready to solve our first reinforcement learning problem. This will be amazing. At first we will define the problem we need to specifiy the environment we want to solve.

## Environment - Grid World
Welcome back in our 4x4 gridworld we already visited last time. Lets's quickly recap the setting.

Imagine we have a 4x4 grid world and the goal of the agent is to find the star. See in the image below.
This environment has 16 states. The start position of the agent is (0,0) and the goal state is (3,3). The task is episodic, which means that if the agent reaches the goal state, the episode terminates. Now everything is set back, and the agents starts again.

![Grid World Setup](/images/grid_world_setup.png)

We define that the agent has 4 different actions (up, down, left or right). This means that if the agent
executes action up in state (0,0) he will end up in state (1,0). If he evaluates an action that he can't do as there is a border, he will
stay in the same state. This means that action left in state(1,0) will lead to state(1,0).

## Monte Carlo Methods
The goal of the agent is to reach the start, which is in the top right corner. We define the reward function as +1 for reaching the star and 0 otherwise. The discount factor $$\gamma = 0.9$$. The influence of the reward function to the
behaviour of the agent we already discussed in the last post.

The agent has no idea about the dynamics of the environment and thus moves accoding to it starting policy $$\pi_{0}$$, which normally results in random behaviour. The agent chooses actions according to this policy and tries to learn a better one. This means to achieve a higher expected return. We will define this more formally in a few minutes.

Let's assume this is the first trajectory $$\tau^{\pi_0}=((0,0),up,0,(1,0),right,0,(1,1),...,(3,2),right,1,(3,3))$$ of the agent. It is shown in blue in the image below.

![Grid World Trajectory](/images/grid_world_trajcectory1.png)

What has the agent learn now? He learned that action right in state (3,2) is good as he got a reward of +1. This
is very obvious. But what about all the other states that are part of this trajectory.

After the first trajectory the agent collected the following information.

$$Q^{\pi_0}((3,2),right) = 1$$

$$Q^{\pi_0}((2,2), up) = 0 + \gamma * 1$$

$$Q^{\pi_0}((1,2), up) = 0 + \gamma * 0 + \gamma^2 * 1$$

$$Q^{\pi_0}((0,2), up) = 0 + \gamma * 0 + \gamma^2 * 0 + \gamma^3 * 1$$

$$Q^{\pi_0}((0,1), right) = 0 + \gamma * 0  + ... + \gamma^4 * 1$$

$$Q^{\pi_0}((1,1), down) = 0 + \gamma * 0 + ... + \gamma^5 * 1$$

$$Q^{\pi_0}((1,0), right) = 0 + \gamma * 0 + ... + \gamma^6 * 1$$

$$Q^{\pi_0}((0,0), up) = 0 + \gamma * 0 + ... + \gamma^7 * 1$$

The agent learned some information about the environment. He learned one path to reach the goal state. If we choose as policy
to always use the maximum q-value in each state, then we would always take the same path. Meaning, the agents exploits it's 
knowledge about a path that leads to the goal. Unfortunately, it's not know if this is the optimal path. 

Looking at our first trajectory we now it's not perfect. It would be better if the agent in state (1,1) takes agent right
instead of going down. But the agent doesn't know that.

To avoid this exploitation-exploration dilemma as discussed in a previous post we need some randomness in mapping states to action.
A simple and common approach is to use an $$\epsilon- greedy$$ approach, where $$epsilon \in [0,1]$$. This means, that we
take the best action with a probabiltiy of $$1-\epsilon$$ and with a probability of $$\epsilon$$ we take a random action. This, supports the agent to continue exploration during training.

After the q-values are updated the agent now has a new policy $$\pi_1$$
Let's assume the second trajectory of the agent looks like shown in the image below.

![Grid World Trajectory](/images/grid_world_trajectory2.png)

The agent now has this information additional to the one from the first trajectory.

$$Q^{\pi_1}((3,2),right) = 1$$

$$Q^{\pi_1}((2,2), up) = 0 + \gamma * 1$$

$$Q^{\pi_1}((1,2), up) = 0 + \gamma * 0 + \gamma^2 * 1$$

$$Q^{\pi_1}((1,1), right) = 0 + \gamma * 0 + \gamma^2 * 0 + \gamma^3 * 1$$

$$Q^{\pi_1}((1,0), right) = 0 + \gamma * 0 + ... + \gamma^4 * 1$$

$$Q^{\pi_1}((0,0), up) = 0 + \gamma * 0 + ... + \gamma^5 * 1$$


The agent now learned that in state (1,1) the action right is better than action down as Q((1,1), right) > Q((1,1), down). But it
still doesn't know if there still exists a better trajectory.

Now, the agent would sample a third trajectory, than a fourth and so on.

During the training the agent visits the same the same state action pairs Q(s,a) multiple times. This could potentially
lead each time to a different return as it depends on the policy that is used for this trajectory. We will average all this values. By the law of large numbers the averages of these estimates converges to the expected return.

This approach leads finally to the optimal state action value and thus our agent can solve the grid world in a perfect way.

This kind of sample based method is called **Monte Carlo Methods** and we will formalise a pseudocode algorithm in the next section.

# First-Visit Monte Carlo Method
In the section before we described an approach to learn the optimal policy by random sampling. These approaches are in general
called Monte Carlo Methods ([Wikipedia Article on Monte Carlo Methods](https://en.wikipedia.org/wiki/Monte_Carlo_method)).

In a trajectory it's possible to see the same state action value multiple times. This means haveing a circle in the states
that have been visited. We need to discuss how we handle these multiple occurences of state action values. 

One easy way is to only use the first visit of an state action value to update it. The corresponding method is called
First-Visit Monte Carlo Method. 

In this method we only look at the first visit of each q-value and update it. Later occurences of the q-value are ignored.

Let's shortly talk through the pseudo code below.

At the beginning we initalize all q-values with an arbritrary value and initalize a epsilon greedy policy.
Each q-value has a list of returns that are used to build the average. At the beginning we not to clear this list.

The following part will be removed until our algorithm converged.

We generate a trajectory $$\tau$$ based on the current policy $$\pi$$ and initalize the return of this trajectory to 0.
Now we start with the next to last state and update the return by the reward of this step and a discounted reward of the future
returns.

If the state action value is the first appearance of this combination than update everything. Meanig to add this value to the list and calculate the average of them. If it is not the first appearance than nothing will be updated yet. 

Now we continue with the previous state.


@ TODO: Convert to latex algorithm
    def CalculateFirstVisitMC():
        Initialise Q(s,a) arbritarily for all s in S and a in A
        Initalise policy as \epsilon-greedy
        Returns(s,a) = empty list of returns for all s in S and a in A

        for episode in episodes:
            Generate Trajectory based on current policy \pi
            Return = 0
            Iterate from the last state in the trajectory towards the first state
                Return = r_{t+1} + \gamma * Return 
                If St,At is first appearance in trajectory
                    Append Return to Returns(S_t,A_t)
                    Q(S_t,a_t) = average(Return(s,a))


## Monte Carlo Methods Summary
This section described how we can use a Monte Carlo approach to solve the grid world with reinforcement learning. The good thing
is that the agent can directly learn from the experiences without the need of a model. But we still need to have a full episode
with a final outcome to learn from the data. A di

# Temporal Difference Learning (TD Learning)
Temporal Difference Learning (TD Learning) solves the same problem as the Monte Carlo Methods. Where MC needs a full episode, TD methods update their estimates based on estimates of other states. Thus, there is no need to wait until the end of a episode.
Updating an estimate without waiting for the end is called bootstrapping. Maybe it still sounds a bit vague. Let's save the our
gridworld with the TD method.


# +++++++++++++++ Unordered Things


Q Learning

Bellmann Equation

Temporal Difference Learning
Monte Carlo Methods
SARSA
On Policy Learning 
Off Policy Learning

The connection between Monte Carlo and TD Learning (n-step TD Learning)

++++ Think about how to create a graph by code. Then there is no need to drag and drop and adapt the things
