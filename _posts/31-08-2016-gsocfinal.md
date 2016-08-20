---
layout: post
title: (GSoC) Improving the State of Optim.jl
author: Patrick Kofod Mogensen
excerpt: The summer is almost over, at least Google Summer of Code
---
# Google Summer of Code 2016: Optim.jl
In the spring, my proposal to work on [Optim.jl](optim-url) was accepted by the [NumFOCUS](num-url) organization and
[GSoC 2016](gsoc-url). It has been a great opportunity for me to
improve my programming skills in general, and work on a Julia package that I care about.
Some things have worked out, and some things haven't.

# Optim.jl
The package I have worked on is basically a collection of solvers with a coherent interface, mainly for
[unconstrained mathematical programming](wiki-opt). Standard
algorithms such as gradient descent (and variants), conjugate gradient, (L-)BFGS, and variations
on Newton's Method.

At the beginning of GSoC'16, the package was actually in okay shape. Lots of the functionality
was in there, and the package generally *worked*. However, there were some long standing issues,
some frozen pull requests, and more to be done. Benchmarking was done on a few select
problems, and some solvers lacked testing.

# My Life as Groundskeeper Willie
The biggest surprise is how much *janitorial* work I have done over the summer.
Optim is, and was, by no means idle or inactive. However, there is always a specific danger
related to an open source project's life: the volunteering developers. No one is getting
paid (unless they're GSoC'ers), so the project only receives the resources (time) that
people can throw at it without jepardizing their family lifes, work lifes, or studies.
Optim saw an initial sprint with a lot of activities, but lately the work has been more
sporadic. Given my activity under the GSoC banner, I have to a large extent worn
one of the "maintaner" caps. This means responding to issues, reviewing pull requests
(including reading the academic work underlying them), tagging releases, handling deprecations
and iteroperability with other packages, and more.

It might seem as if the experience has taken time and effort away from the things
in my original proposal, and obviously it has. However, I really think it has given me a great insight
into the way open source communities work, and what it takes to successfully develop and
maintain open source software. While it is only a hypothesis, I dare to suggest
that the increased activity coming from the GSoC-project has increased general interest
of contributing to the package. New contributors have made successful pull requests
or important things such as: improved support for preconditioning, a Newton solver
using a trust region approach to ensure convergence, a particle swarm solver,
and more.

# Pushing a polytope downhill
A major re-write of the Simulated Annealing solver was also carried out as
part of this project. The old solver was essentially a direct implementation of
the original Nelder and Mead (1965) paper. As shown in Gao and Han (2010)), it is
possible to improve on the performance for high-dimensional problems, by letting
the parameters vary with the dimension of the input to the objective function.

The pull request (merged) can be found [here](neldermead-url), and documentation
of the new functionality is [here](neldermead-doc). Besides improving the ability
of finding the minimizer, the new code is also significantly faster, and has
a much smaller memory use. In an application mentioned in the pull request it is a
hundred times faster, and requires on 1/20th of allocated memory as the old version.

# Benchmarks
The state of benchmarking in Optim was lacking. Over the summer I have experimented
with testing the solvers on the well-known [CUTEst](cutest-url)
test set. I leveraged the existing package [CUTEst.jl](cutestjl-url)
to easily compile and call the objective functions and gradients for each problem.
The package has been a bit of a moving target, as it has seen some changes over the summer,
but the development branch is now in a state where it is much easier to use from Optim's side.

Some pictures showing the results can be seen below.
![][cutest-f]
![][cutest-x]
![][optim-f]
![][optim-x]

The work is not entirely done, mostly because it is not yet clear what the best hosting
solution is. The benchmarks creates two csv files and a few images per commit that
is benchmarked, and it might be optimal to move this machinery to an OptimExtras.jl
package. The (currently) active pull request can be found [here](benchmark-url).

# What, why, and how?
Prior to the summer, Optim's documentation system was basically the README.md file
in the Github repository. The [new documentation](newdoc-url) is based on [Documenter.jl](documenter-url).
Documenter.jl is a Julia package that generates documentation based on files provided,
and doc strings from the actual code. It is quite need, and we need to take advantage
of more features in the future, for example the automatic testing if the examples provided
actually run, or break due to changes in the code.

The new documentation contains the old information from the README, but also additional information on
particular solvers, how to use automatic differentiation, and more. While the pull
request is merged, more information is still waiting to be added, and hopefully the
documentation can be supplemented with example notebooks in the future.

# Testing
This part of the proposal could have gone further. Testing a solver is not necessarily
straight forward. One thing you can do, as we do, is to take a test problem with a known
solution, and see if the solver succeeds. Testing all the components is a bit more tricky,
although of course possible. More work needs to be done, but so far what was done
was simply to make sure that solvers are tested on the small sample we already have
in Optim, as well as running the benchmarks to see if some solvers seem to have
significant, and surprising, performance issues. The AcceleratedGradientDescent solver
might be such a case identified during this GSoC.

# Misc
Besides the things mentioned in the project description other smaller additions made
it into the package over the summer. A NEWS.md file was added so it is easier to
remember and communicate changes since last version tag. I also changed the automatic
differentiation to use ForwardDiff instead of the DualNumbers we used to use. The most
important improvement here is the ability to use automatic differentiation for the second
order methods, that is we now provide AD Hessians.

# That's All Folks!
I would like to thank Miles Lubin who encouraged me to apply, and has been my mentor,
NumFOCUS which is the organization my project is carried out for, Google for creating
Google Summer of Code, and the Danish weather for providing lots of computer time by
raining all summer.

# List of Commits
To fulfill the requirements, I list all commits below. My commits to Optim.jl can
be found [here](pkofod-commits), but they also include older and will include future commits
done outside of GSoC.

## Documentation
- [24ad090](https://github.com/JuliaOpt/Optim.jl/commit/24ad0905e56efd0f5622b5e9b3dd33eb16547ea7) - getting the docs up and running (introduce Documenter.jl to Optim.jl)
- [c0d2d80](https://github.com/JuliaOpt/Optim.jl/commit/c0d2d809ce4bd1e7c19869b2671bcad7536e30d5)
- [013d264](https://github.com/JuliaOpt/Optim.jl/commit/013d264b88a8d407d85d602b613f7337ade35ae3) - clean README.md of information moved to docs, and improve docs
- [21bd318](https://github.com/JuliaOpt/Optim.jl/commit/21bd318c1bf6a6f08779b67e64a852e241ebadd4) - fix encryption for Documenter.jl
- [025c8cc](https://github.com/JuliaOpt/Optim.jl/commit/025c8cc6417ccc16ce540249115235e9912e32c0) - small Documenter.jl fix due to Julia slightly changing to v0.6-dev

## Features
- [d80282c](https://github.com/JuliaOpt/Optim.jl/commit/d80282c2a85ee2ec2ad3d353f8930ca880dff2b8) allow for more optimizers in combination with box constrains
- [98e50c8](https://github.com/JuliaOpt/Optim.jl/commit/98e50c8e463a5d600fb505a0e1623bb5ac527ecb) - improve Nelder-Mead trace, and parameterize OptimizationTrace to allow for solver based dispatch
- [4e16fb8](https://github.com/JuliaOpt/Optim.jl/commit/4e16fb84026f0800438e31449453f89b04c4da2d) - implement Adaptive Parameters Nelder-Mead and document it
- [88f8e48](https://github.com/JuliaOpt/Optim.jl/commit/88f8e48b3888fe43d7456c4b584a28ff35fff585) - switch to the ForwardDiff automatic differentiation functionality
- [fffb8ca](https://github.com/JuliaOpt/Optim.jl/commit/fffb8cae815b696e5dbe9c3b33fffbe3ff044e09) - allow for automatic differentiation in NewtonTrustRegion

## Testing
- [b1c7de4](https://github.com/JuliaOpt/Optim.jl/commit/b1c7de46b28e9011585df977d23147b71ee9bd69) - trace tests for BFGS and NelderMead
- [b2188d7](https://github.com/JuliaOpt/Optim.jl/commit/b2188d7be5a74fecb0c7a7318df1d589210b4250) - trace tests and cleaning of some files
- [c6a5efc](https://github.com/JuliaOpt/Optim.jl/commit/c6a5efc6c1874955897a43278ccce352cb211c6e) - improve tests for BFGS, GradientDescent, MomentumGradientDescent, Newton, NewtonTrustRegion, and add tests for bounded, univariate optimization

## Misc. changes including general bug fixes
- [b013257](https://github.com/JuliaOpt/Optim.jl/commit/b01425755c653ff8fe11cc1e3aa41b648331f6dd) - preparing transition from Optim v0.4 to v0.5
- [2b586e9](https://github.com/JuliaOpt/Optim.jl/commit/2b586e9403e81a0b0310d2ed184208ead87762de) - add a NEWS.md to keep track of progress in Optim.jl
- [244d032](https://github.com/JuliaOpt/Optim.jl/commit/244d032eee487101268a2e683643c96122034421) - small indentation adjustment
- [5a8e2a1](https://github.com/JuliaOpt/Optim.jl/commit/5a8e2a1bbb58f483039b0ba5b606f76a11b377cf) - use vecnorm instead of norm
- [276c98d](https://github.com/JuliaOpt/Optim.jl/commit/276c98d3a61432a14fad388debce91d056653b88) - Deprecate old NelderMead API
- [32d6b37](https://github.com/JuliaOpt/Optim.jl/commit/32d6b37a8bc5764ac75209fa44d3b40f92a75382) - NEWS.md coverage improvement
- [3351bb6](https://github.com/JuliaOpt/Optim.jl/commit/3351bb604723991d6ba86d33d9610fff5ccfe849) - make use of syntax for lower and upper bounds constraints consistent
- [bb81e1e](https://github.com/JuliaOpt/Optim.jl/commit/bb81e1edbf1ba2b5c2ca450aa8ca5a63499d1b2b) - cleaning of NewtonTrustRegion and ParticleSwarm constructors and NEWS.md
- [e8e75a8](https://github.com/JuliaOpt/Optim.jl/commit/e8e75a89790ab49305d4d07aa30b762ce93a25d2) - fix direction reset branch in ConjugateGradient
-[2e7d238](https://github.com/JuliaOpt/Optim.jl/commit/2e7d2388e12d1e3b555bd4c72a2962018ea553b6) - fix small NelderMead printing error
- [d25efee](https://github.com/JuliaOpt/Optim.jl/commit/d25efee438a0ce15c371392b2155d10af0bc4640) - get package ready for v0.6.1

[optim-url]: https://github.com/JuliaOpt/Optim.jl
[num-url]: http://www.numfocus.org/
[gsoc-url]: https://summerofcode.withgoogle.com/
[wiki-opt]: https://en.wikipedia.org/wiki/Mathematical_optimization
[cutest-url]: http://www.cuter.rl.ac.uk/Problems/mastsif.shtml
[cutestjl-url]: https://github.com/JuliaSmoothOptimizers/CUTEst.jl

[cutest-f]: https://pkofod.github.io/files/cutest-f.png
[cutest-x]: https://pkofod.github.io/files/cutest-x.png
[optim-f]: https://pkofod.github.io/files/optim-f.png
[optim-x]: https://pkofod.github.io/files/optim-x.png

[benchmark-url]: https://github.com/JuliaOpt/Optim.jl/pull/259
[neldermead-url]: https://github.com/JuliaOpt/Optim.jl/pull/220
[neldermead-doc]: https://juliaopt.github.io/Optim.jl/
[documenter-url]: https://github.com/JuliaDocs/Documenter.jl
[newdoc-url]: http://www.juliaopt.org/Optim.jl/latest/
[pkofod-commits]: https://github.com/JuliaOpt/Optim.jl/commits/master?author=pkofod
