# What is PID resampling?

## What is PID?

We are using the PID variables to describe how much the particle in a track behaves like a certain particle type. They tell us things like: "This particle looks much more like a Kaon than a Muon". If we then select for particles that look very much like Kaons, we get a sample where most of the particles are actually, in fact, true Kaons. This is obviously very useful. To give an example, let's say that `particle_PIDp` is a measure for how much the `particle` looks like a proton. The higher the value, the more the algorithm thinks it is a proton.

## What is the problem?

The PID information is calculated by an algorithm that takes into account various features of the event. The problem is that in our simulated data ("Monte Carlo"), we are currently still unable to simulate all of these features precisely enough. That means that the PID information we calculated on simulated data is incorrect. We are usually a bit to good in recognizing the particle: For a particle that is simulated to be a proton ("true proton"), the `proton_PIDp` tends to be too high.

This means that if we try to measure the efficieny of, for example, a cut on the `proton_PIDp` on Monte Carlo, we are getting it wrong. And if we use the PID as an input for a multivariate classifier, we are getting an incorrect response.

![problem.png](https://raw.githubusercontent.com/KonstantinSchubert/PID_resampling_text/master/problem.png)

## What does PID resampling do?

The trick is to throw away the PID variables in our Monte Carlo samples (only there) and create new ones. This is possible because it turns out that  the PID response (`particle_PIDp`,`particle_PIDK`, ...) is approximately a function of
   1. The true particle type
   2. The particle momentum
   3. The event multiplicity (how many charged tracks are in the momentum)
   4. The pseudo rapidity of the particle

Another way of saying this: For a give true particle, the correct PID response is highly correlated with the three kinematic variables `P`, `ETA` und `nTracks`. The good thing is that we know the true particle type in Monte Carlo (because that's how we simulated it) and the other 3 variables are reasonably well simulated as well.

![correlation](https://raw.githubusercontent.com/KonstantinSchubert/PID_resampling_text/master/correlation.png)

In order to obtain the correct relation between these 3 kinematic variables and the PID response we need a "clean" data sample where we have all particles correctly identified. (Without cutting on the PID). In this data sample, all particles reconstruced as, say, Kaon, must actually be true Kaons. It is the task of the PIDCalib group to provide these data samples. There are special samples from different decays for each particle type. We bin each of these data samples in bins of `P`, `ETA` and `nTracks` of the corresponding particle.

Then we can resample. For each event in Monte Carlo:
   1. We look at the `P`, `ETA` and `nTracks` variable of the particles who's PID we want to resample.
   2. In the data sample, we find all events that are in the corresponding bin of `P`, `ETA` and `nTracks`.
   3. These events show a distribution in each PID variable that is typical for the bin.
   4. For each PID we want to resample, we draw a PID value from its distribution and take it as the resampled PID for the MC event.

# When should I (not) use it?
  (Somebody please confirm the following)
  * Use it to get a feel for the PID response in your signal channel
  * Use it to  quickly optimize your preselection.
  * Use it (if you're confident) to train a multivariate classifier (no warranty).
  * Don't use it to calculate the efficiency of a single orthogonal cut. For reasons listed below, is more precise to direcly measure the efficiency on data for each kinematic bin.

# When PID resampling fails

## Negative bin weights

In order to remove the remaining underground in the clean data samples, the PIDCalib group calculates s-weights (can somebody point me to a good reference for this) for each event. This "weights away" the remaining background from the PID distributions. However, because S-weights can be negative and because the conditions for a successful S-weighting can not always be fulfilled, it can happen that in some bins of `P`, `ETA` , `nTracks`, a PID distribution may look like this:

figure goes here!!!!

There are quite a few negative bins. In order to randomly draw from this distribution when resampling, one has no other choice (?) than to set these negative bins to zero. Even if these bins would have in fact been zero if there was no background, we must assume that in other kinematic bins there were upward fluctuations the bin content of the bins which are here negative. In total one creates a bias where the resampled bins in the resampled distribution are too high.


This effect can be reduced if we just want to know the efficiency of a cut. In this case we integrate the s-weighted number events up to the cut point and divide by the full norm. I dont't understand s-weights enought to make qualified statements about this, but while I think it doesn't fully resolve the underlying issue, it does certainly mitigate it to a certain degree.
