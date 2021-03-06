

# SIR CTMC inference example

Andrew Black, 9/06/2020

This is an example of how to run the inference. More details of the algorithm
will be given somewhere else, but the main idea can be found in the paper [Computation of epidemic final size distributions.
J. Theor. Biol., 367, 159-165](https://dx.doi.org/10.1016/j.jtbi.2014.11.029)
and involves calculating the likelihood by solving the forward equation for the process. 

We then use the the AdvancedMH package to do the MCMC. 

~~~~{.julia}
using Random, Distributions
using AdvancedMH, MCMCChains
using Plots, StatsBase, StatsPlots
include("SIR_noQ.jl")
~~~~~~~~~~~~~


Simulate some test data
~~~~{.julia}
Random.seed!(7)
data = SIR_sim(100,2.0/3,1/3)

Z1_obs = data[1][1:end-5] # truncate after last obs.
Z2_obs = data[2][3:end]   # truncate until first obs.
plot(data[1], label = "infections", legend=:bottomright)
plot!(data[2],label = "recoveries")
xlabel!("time (days)")
ylabel!("cumulative number of events")
~~~~~~~~~~~~~


![](figures/SIR_examples_2_1.png)\ 




This version uses the number of infection events on each day.

![](figures/SIR_examples_3_1.png)\ 




Setup the likelihood function and proposal distribution. 
The covariance matrix was determined from a pilot run

~~~~{.julia}
function density(theta)
    if  theta[1] < 0 || theta[2] < 0
        return -Inf
    else
        return likelihood_tight(theta[1]/theta[2],1/theta[2],Z1_obs,100)
    end
end

C = [ 0.130982  0.190005; 0.190005  1.0076]
rw_prop = RWMH(MvNormal(0.5*C))
model = DensityModel(density)
chain = sample(model, rw_prop, 10^4; param_names=["R0", "1/\\gamma"], chain_type=Chains, init_params=[2.5,2.0]);
plot(chain)
~~~~~~~~~~~~~


![](figures/SIR_examples_4_1.png)\ 

