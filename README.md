# GSoC Final Report

This summer I have been working with Patrick to improve packages in `JuliaNLSolvers` family, especially for `LsqFit.jl`. I'm extremely thankful for GSoC and all the support I received from Partrick and Julia community to help me get through this project and attend JuliaCon London.

## Summary

At this part I will summarize the main work I've done.

### Part 1: Documentation
Documentation has always been important for users, and could never be enough.

`Optim.jl` has good documentation but lacks some examples. I built two notebooks using `Optim.jl` to show the usage of maximum likelihood and optimization trace. These notebooks can be found in `/Notebooks`.

![trace](/trace.png)

`LsqFit.jl` is the most basic package in `JuliaNLSolvers`, the documentation is the `README.md` in the GitHub page and covers only the usage of functions. I made a documentation covering introduction, getting started, and tutorials to help users understand how and why behind the code. These documentation is generated using `Documenter.jl`. The source code and updated `README.md` could be found in `/LsqFit.jl/docs` and `/LsqFit.jl`. Part of the documentation is [online](https://julianlsolvers.github.io/LsqFit.jl/latest/) since some codes have not been merged.

### Part 2: Functionality
I added more functionalities for `LsqFit.jl` to fix the error weight problem, assess goodness of fit, show fitting results in an elegant way and more algorithms.

There is mistake in the weighted calculation of `LsqFit.jl` and I proposed the [fix](https://github.com/JuliaNLSolvers/LsqFit.jl/pull/74). But now I think it should be handled using keyword argument.

Now `LsqFit.jl` could asssess goodness of fit by using following functions:

- `mse(fit)`
- `sse(fit)`
- `sst(fit)`
- `r2(fit)`
- `adjr2(fit)`

The `fit` result is now printed as:

```julia
# fit data
>julia fit = curve_fit(DoseResp, xdata, ydata, initial_p)

# output
Results of Least Squares Fitting:
* Algorithm: Levenberg-Marquardt
* Iterations: 8
* Converged: true
* Estimated Parameters: [0.178863, 1.00522, -5.82878, 0.830257]
* Sample Size: 9
* Degrees of Freedom: 5
* Weights: Float64[]
* Sum of Squared Errors: 0.0007
* Mean Squared Errors: 0.0001
* R²: 0.9991
* Adjusted R²: 0.9983

Variance Inferences:
k   value std error     95% conf int
1  0.1789    0.0116   (0.149, 0.209)
2  1.0052    0.0145   (0.968, 1.043)
3 -5.8288    0.0321 (-5.911, -5.746)
4  0.8303    0.0519   (0.697, 0.964)
```

The working version of these fetures is in [`curve-fit-tools`](https://github.com/iewaij/LsqFit.jl/tree/curve-fit-tools) branch and `/LsqFit.jl/utilities`.

I also worked on adding two more algorithms:

- Gauss-Newton
- Steepest Descent

The building of algorithms motivate the reconstruction work in part 3 since we need more abstractions and invlove linesearch. The reconstruction work is still in progress and therefore algorithms need wait till the reconstruction work finishes to be added. The rough work could be seen in `/LsqFit.jl/solvers`.

### Part 3: Reconstruction
The main goal of the reconstruction is to keep the same interface as `Optim.jl` and involve functionalities from `NLSolversBase.jl` and `LineSearches.jl`. It will also provide more abstractions for solvers. Gauss-Newton and Steepest Descent method will be added after the reconstruction finishes.

There will be a new function `least_squares()` which behaves similar to `optimize()` but accept only least squares algorithms. `curve_fit()` will then keep the same interface. For example, to pass `x_tol`, the code will be:

```julia
curve_fit(m, tdata, ydata, p_init, LsqFit.Options(x_tol = 1e-8))
```

To pass algorithm's parameters, the code will be:

```julia
curve_fit(model, tdata, ydata, p_init, LevenbergMarquardt(min_step_quality = 1e-3))}
```

We can also use different automatic differentiation method:

```julia
curve_fit(m, tdata, ydata, p_init, autodiff = :forward)
```

This has been the most challenging work so far. `NLSolversBase` assumes the objective to be 1-d for `Optim.jl` or squared-shape for `NLSolve.jl`, but not rectangular-shape residual function for `LsqFit.jl`. Some functions needed have not yet supported for `LsqFit.jl`. I submitted several PRs for `NLSolversBase`, [#88](https://github.com/JuliaNLSolvers/NLSolversBase.jl/pull/88) and [#90](https://github.com/JuliaNLSolvers/NLSolversBase.jl/pull/90), to fix these issues.

There are a lot of bugs, because the reconstruction involves too many changes at the same time. And when the reconstruction collide with Julia upgrating to v1.0, there are even more bugs. The reconstruction is still in the progress of debugging. The rough work could be seen in `/LsqFit.jl`.

## Challenges
- It is difficult to imagine what users need and what difficulties users are facing. For example, in the discourse [post](https://discourse.julialang.org/t/fitting-dose-response-curves/12856), users are finding difficulty in the model definition, which I never thought of.
- Tracing bugs and errors in Julia is hard. Hopefully `Rebugger.jl` will make it a lot easier.
- My original proposal involves too many packages in different problems that I lose focus.

## Future Work
I'll continue working after GSoC ends since I started late. The plans include:

- Continue the reconstruction (debugging) and algorithm work.
- Adding plot and bootstrap functionalities in `LsqFit.jl`.
- A lot of interface problems need to be discussed, for example, the `Options()` and `wt` argument.
- Benchmarks against other packages and languages.
