## Using Reinforcement Learning to make rockets land

This project uses artificial intelligence algorithms, and more precisely deep reinforcement learning algorithms, to make an agent learn by itself how to land an orbital class rocket, Starship. The agent is the algorithm which can observe the environment and choose actions to complete some task, here, landing the rocket. Before the so-called "training", the agent just go initialized and basically chooses actions at random. However, as the training continues, the agent gets reward for doing the task we want it to do : again, landing the rocket. Based on this reward and how he got it, the agent will update the parameters of its neural network in order to choose actions that led to this reward more probable.

### What is Starship ?

Starship is a fully reusable launch vehicle which is currently being developed by SpaceX. Starship refers to two things : the rocket as a whole (ie the first stage, named Super Heavy, and the second stage) as well as the second stage itself. In this project, Starship refers to the second stage. This whole next-gen rocket, with its crazy diameter of 9 meters, will be capable of delivering more than 100 tons to Low Earth Orbit, while having a reusable first and second stage. Starships are currently being built in Boca Chica Village, Texas, and, maybe one day, will bring humans to Mars.

The second stage of the rocket, Starship, will perform a special reentry profile to slow down enough as shown below (it will be coming from orbit so will have a high speed w.r.t the air surrounding it) : it will keep during its descent a high Angle Of Attack (60°) and then will fall like a sky diver, belly first. A few seconds before touchdown, it will executes a maneuver to bring it from belly flop to tail down to actually land vertical, on its landing legs.



### What is Reinforcement Learning ?

Reinforcement learning is a subfield of AI that enables autonomous agents to learn by trail and error. RL allows agents to interact with an environment (here, the physical environment with the rocket, the landing platform etc...) and learn from this interaction. The learning is done with rewards, which will tell the agent what task we want it to perform. In this case, the agent receives a reward of +1 for successfully landing the rocket, upright at almost no speed. If it fails, then it receives no rewards. Based on the rewards it receives, the agent will then deduce which actions are good to take (for example, firing the main engine near the landing pad to slow down) and which actions are bad (for example, firing the side thrusters when the rocket is perfectly upright). One last thing : the agent, in order to perform the right actions at the right time, must know where is the rocket, what is its orientation, its speed and more. We thus give him, at each timestep (each time he must take an action), an observation of the environment. In this project, the agent is given at each timestep 11 observations, going from position, speed to rotation of the vehicle.

The adjective "deep" refers to the fact that the algorithm used here uses "deep" neural networks, ie. neural networks with multiple hidden units. This "deepness" allows the agent to learn complex relationship between the agent to choose and the state the agent sees.

This learning is done, in this case, using the PPO algorithm, developed by OpenAI, and implemented by the Unity team in Python. Learn more about the deep RL implementation in the Technical Details section.



## Discussion, and Technical Details

- ### About the physical simulation and the landing

  TODO : decaller le tout non?

  ​	In reality, to achieve a landing, Starship will essentially be able to control the throttle level of its 3 engines, the gimbal of these engines, the RCS side thrusters and the position of its body flap and top fins.

  ​	In the simulation I created, the atmosphere was not considered and thus the agent doesn't get to control the position of the flaps. Furthermore, I chose to only provide to the agent one throttle level, firstly because then the RL problem was a discrete action problem, and also because from physical point of view, achieving landing with the full throttle is optimal (more on that later). Apart from that, the agent lands the Starship using TVC (engine gimballing) with a 7° angle (that number comes from what the Merlin engine is capable of on the Falcon 9's first stage). The agent can of course turn off and on the 3 main engines, as well as a total of 4 RCS thrusters, to help stabilize the orientation of the rocket. With the combination of TVC and RCS, the agent can thus control the roll and pitch of the rocket (the yaw control is neglected)

  ​	At the start of the simulation, the rocket spawns at an altitude of about 5 km, with a random initial velocity and in the belly flop position (again, with a bit of random initialization). The center of mass of rocket is set at about 2/3 from the top of the rocket. A landing is considered successful if the speed of the rocket at touchdown is less than 5m/s and the rocket stands upright.

  ​    Interestingly, no matter the reward function used, the agents always converged on one special (and very efficient) landing profile : the suicide burn, or, as SpaceX calls it, the hoverslam. This kind of maneuver, well know of KSP players, is simple : fire the main engine to slow down at the very last moment. As pointed out by Scott Manley (@DJSnM) in his video (https://www.youtube.com/watch?v=T3_Voh7NgDE), this maneuver, while being dangerous, is the most optimal one to land considering the fuel it requires. For those wondering, yes the reward function only rewards the agent of successfully landing (regardless of the fuel it used for landing), but the overall learning algorithm tries to maximize the reward per timestep. So for example, during 100k timesteps, if you use a safe landing method, then you will be able to achieve less landings than if you used the hoverslam technique, because it is way more fuel as well as time efficient.

  ​	The first limitation of this simulation is of course that it doesn't take into account the atmosphere. It follows that precise trajectories and speeds are not followed. Also, another limitation is that the agent is able to turn off and on the 3 main engines instantly (1) and for an unlimited number of times (2), and same for the RCS. This does not really reflects reality as the main engines are complex machines which (1) doesn't start instantly and (2) can't restart for a large number of times. 

  BELLY FLOP MANEUVER ?

- ### About the Deep Reinforcement Learning implementation

  ​	The Reinforcement Learning problem associated with the Starship landing is a continuous state space and discrete action space problem. As stated above, the agent receives at each timestep 11 observations which are floating point values associated with the position, rotation and speed of the rocket. The agent then acts and chose, again at each timestep, among 162 (= 2x3^4) possible actions. While the number of actions seems large, the agent only controls the main engines throttle level (2 actions: on or off), the main engine TVC on the X (3 actions: -7°, +7° or no TVC) and Z axis (3 actions: -7°, +7° or no TVC) and finally the roll RCS (3 actions: negative, positive, no RCS) and pitch RCS (3 actions: negative, positive, no RCS).

  ​	The reward function, as stated above, is very simple as the agent only get a reward of +1 for achieving a successful landing. However, having such a simple reward function is critical at the beginning of the training : executing random actions (what the agent does at the beginning) have a very low probability (1 in 8e441, which is a 8 followed by a good number of 0's) of leading to a positive reward (the agent can't learn anything if during all his training he saw no rewards). Having the agent actually reach the landing state is thus crucial to "get in on tracks". To do that, I used a combination of 2 techniques : curriculum learning and imitation learning. Curriculum essentially make the agent learn a simpler task than the task we finally want it to perform. For example, instead of spawning the rocket at an altitude of 5 km, the rocket is spawned at only 500 m. This makes the agent much more closer to the landing state, giving him more chance to reach it and observe a reward. Imitation learning is essentially providing examples to the agent about what to do in order to achieve its goal. For that, I actually recorder myself landing the rocket, which was still possible at low altitude with no rocket orientation (don't even try to land it yourself on the final levels, that's just impossible, or should I say, very very very hard). This technique is often considered as sub-optimal because we influence the learning of the agent, providing him examples which may not be optimal. However, in this case, I wouldn't call it imitation learning as it was just used for the very beginning of the training and then discarded after some fixed amount of timesteps (the agent trained using my demonstrations during 0.0025% of its training). The agent was then able to learn the optimal sequence of actions to perform, with no influence of my demonstrations.

  ​	So at the beginning, the agent was guided by human demonstrations so that it experienced the positive reward. We can kind of see this as an exploration strategy : instead of exploring using completely random actions, the agent explores the environment with actions that are overall close to actions which lead to a reward. This and the curriculum seems to be a good strategy in settings like these, ie. where the reward function is very sparse. Of course, this type of imitation learning cannot be used in every environment as well as curriculum learning, indeed, it may be difficult for some tasks to be reached for simpler tasks.

  ​	In order to get agent that are able to generalize well to different conditions, random initialization was used intensely during all the training. This had the positive of effect of making the agents more robust to variations in changes of initial condition. A direct effect of this could be observed during the training, where the altitude was incrementally increased (from 500m to 5km). At each increase (eg. from 900m to 1.5km), I could observe that the performance of the agent didn't drop by very much (less than expected considering the fact that the spawning altitude was considerably changed).



et en vrai ? c'est ça qui est utilisé?

+parler des models fait dans Blender ?

du son ?



other work ?



## Results

une partie résults, ce qui a marché et ce qui a moins bien marché avec les hyperparams. (+NN)



