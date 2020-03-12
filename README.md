# General epidemic simulation with super-spreaders, extinguishing events, quarantine, and multiple outbreak centers
 by Roger Ison, roger@miximum.info

  Outbreaks are represented with a reasonably standard "SIR" (Susceptible, Infected, Recovered) model.
The idea is that you have an initial population of uninfected but susceptible individuals, and a small
number of initially infected individuals. Disease spreads from each infected indivdidual to some number
of susceptible ones at a rate R0. If R0 > 1 the outbreak will explode; if < 1 it will extinguish. 
The simulation proceeds in cycles of fixed duration. Some number of infected cases die; the rest recover.
Recovered individuals can't be re-infected.

  Note that R0 is the "naive population" spread rate, when almost everybody is uninfected. When herd immunity
is enabled in the model, then the spread rate is reduced according to the population proportion of uninfected indivduals.

  The model proceeds in cycles, discussed below. On each cycle, the currently infected individuals infect
some number of new cases, while going on themselves to either recover or die. If modeling multiple, simultaneous
outbreaks, the cycles of all the outbreaks are synchronized as simulated time proceeds.

## Modeling super-spreaders
  This simulation goes beyond basic exponential math by modeling "super-spreaders". While R0 is the average
transmission rate, for every infected individual we draw a spread rate from a log-normal distribution having a
mean of R0 with some specified dispersion. As a result, some infected cases spread to a large number of new victims,
while others spread virtually not at all - that is the nature of log-normal distributions.
Therefore, the outcome of a simulation is stochastic in the log-normal spread rate. The sim runs multiple 
trials with the same parameters to get an idea of the range of possible outcomes.

  See here:  https://en.wikipedia.org/wiki/Log-normal_distribution  to learn about log-normal distributions.
 
## Modeling extinction events
  The model allows to specify an "extinction event" which occurs at a specified cycle number. The 
extinction event, if present, changes the underlying spread rate R0 to a new, specified number. If it is
less than 1.0, the epidemic will eventually die out. Extinction events might include an effective medication;
introduction of a vaccine; or a partially effective quarantine.

  Of course, if the model includes herd immunity then the outbreak will also extinguish before everyone gets
infected. In this case, the extinction event need not reduce R0 below 1.0; even a small positive number like
1.1 will still yield improved outcomes.

## Modeling herd immunity
  If this feature is enabled in the model parameters, then the effective spread rate will be reduced as the
number of recovered individuals grows. The scale of reduction of R0 is the proportion of uninfected individuals
to uninfectable individuals. 

## Multiple outbreak centers
  The model supports creating multiple, communicating outbreak centers in geographically dispersed populations.
A small proportion of population is modeled to move between each pair of centers. Outbreak centers can be created 
either randomly in the area of a 2D circle around the initial, root outbreak; or along a line, with the spacing 
between centers increasing by powers of 2.

  The proportion of population who travel, or the probability of travel in each direction, is defined as an 
exponentially declining function of distance; the decay rate or scale of distahnce is a parameter of the simulation.
In the model, the proportion of individuals who travel is selected; some are infected, some are not. The infected
individuals contribute to spread where they visit; the uninfected ones may become infected where they visit, and
then return home. 

  Because the number of travelers is generally small, infected travelers are not assumed to spend
the entire cycle where they visit; they may contribute to infection both at home and abroad. This could be 
regarded as an imperfection in the simulation's bookkeeping. Moreover, no accounting is made as to whether
traveling individuals might visit more than one other outbreak center. These bookkeeping details are probably
unimportant to the final outcome. 

  The purpose of this feature is not to emulate any specific geographical map, but to observe the delays 
between outbreaks, and other pandemic behaviors, and also the effects of regional quarantines (which are not
as effective as one might hope).

## Modeling travel quarantine
  The model allows imposition of a strict quarantine that blocks travel between geographically separated
outbreak centers. This is intended to help understand the effect of such policies. The quarantine does NOT
operate within individual outbreak centers; use the extinction events for that.
  
  By playing with the simulation's basic parameters, you can get a numerical sense of how a real epidemic might 
play out. If you adjust them to track reported case numbers, you can estimate how long it might be before burn-out,
how many cases and deaths may occur, and the impact of a vaccine coming available.

# Structure of the simulation

  Each outbreak is represented by an "Outbreak" class object. Being Python objects, outbreaks execute slowly compared
to compiled code. An outbreak begins with a fixed population of 10 million unless you edit the code. One outbreak is the ROOT, 
where initial infection occurs. All other outbreaks start with no infections. 

  Each step of an outbreak is implemented by calling the OneCycle() method. OneCycle() does the bookkeeping of generating
new infections from the current active infections, and recovering or killing off the current active infections.
The number of individuals in the uninfected, infected, recovered, and dead pools is recorded after each cycle step.
For multi-center outbreaks, each cycle has two phases. First, the number of travelers is determined, both infected
and uninfected. These are recorded in Outbreak object local variables. Then OneCycle() determines
the new numbers of individuals in the SIR pools. During multi-center simulations, OneCycle() calls other methods to
pass traveler infection consequences back to the various Outbreak objects.

  This architecture is a compromise between agent-based modeling and simple differential equations. An agent based
model would have many more parameters and distributions, such as the likelihood that one agent will encounter another
during a cycle. The more parameters a model has, the harder it is to "tune" to the real world, and the harder it is
to understand. In the end, an agent model must be run millions of times over a range of parameters, to understand its
expected behavior and variability.

  The model here only deals at the individual level in one place: where, for every infected individual, the model
draws from a log-normal distribution to determine the number of people that person infects for the next cycle. This is
fast and gives a comprehensible and intellectually defensible result.
