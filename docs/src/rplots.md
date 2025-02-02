# Recurrence Plots

## Recurrence Matrices

A [Recurrence plot](https://en.wikipedia.org/wiki/Recurrence_plot) (which refers to the plot of a recurrence matrix) is a way to quantify *recurrences* that occur in a trajectory. A recurrence happens when a trajectory visits the same neighborhood on the phase space that it was at some previous time.

The central structure used in these recurrences is the (cross-) recurrence matrix:
```math
R[i, j] = \begin{cases}
1 \quad \text{if}\quad d(x[i], y[j]) \le \varepsilon\\
0 \quad \text{else}
\end{cases}
```
where $d(x[i], y[j])$ stands for the _distance_ between trajectory $x$ at point $i$ and trajectory $y$ at point $j$. Both $x, y$ can be single timeseries, full trajectories or embedded timeseries (which are also trajectories).

If $x\equiv y$ then $R$ is called recurrence matrix, otherwise it is called cross-recurrence matrix. There is also the joint-recurrence variant, see below.
With `RecurrenceAnalysis` you can use the following functions to access these matrices
```@docs
RecurrenceMatrix
CrossRecurrenceMatrix
JointRecurrenceMatrix
```

## Advanced recurrences specification

```@docs
AbstractRecurrenceType
RecurrenceThreshold
RecurrenceThresholdScaled
GlobalRecurrenceRate
LocalRecurrenceRate
recurrence_threshold
```


## Simple Recurrence Plots
The recurrence matrices are internally stored as sparse matrices with Boolean values. Typically in the literature one does not sees the plots of the matrices  (hence "Recurrence Plots"). By default, when a Recurrence Matrix is created we "show" a mini plot of it which is a text-based scatterplot.

Here is an example recurrence plot/matrix of a full trajectory of the Roessler system:
```@example MAIN
using RecurrenceAnalysis, DynamicalSystemsBase

# Create trajectory of Roessler system
@inbounds function roessler_rule(u, p, t)
    a, b, c = p
    du1 = -u[2]-u[3]
    du2 = u[1] + a*u[2]
    du3 = b + u[3]*(u[1] - c)
    return SVector(du1, du2, du3)
end
p0 = [0.15, 0.2, 10.0]
u0 = ones(3)
ro = CoupledODEs(roessler_rule, u0, p0)
N = 2000; Δt = 0.05
X, t = trajectory(ro, N*Δt; Δt, Ttr = 10.0)

# Make a recurrence matrix with fixed threshold
R = RecurrenceMatrix(X, 5.0)
recurrenceplot(R; ascii = true)
```
```@example MAIN
typeof(R)
```
```@example MAIN
summary(R)
```


The above simple plotting functionality is possible through the package [`UnicodePlots`](https://github.com/Evizero/UnicodePlots.jl). The following function creates the plot:
```@docs
recurrenceplot
```


Here is the same plot but using Unicode Braille characters
```@example MAIN
recurrenceplot(R; ascii = false)
```

As you can see, the Unicode based plotting doesn't display nicely everywhere. It does display perfectly in e.g. VSCode, which is where it is the default printing type.

## Advanced Recurrence Plots
A text-based plot is cool, fast and simple. But often one needs the full resolution offered by the data of a recurrence matrix.

There are two more ways to plot a recurrence matrix using `RecurrenceAnalysis`:

```@docs
coordinates
grayscale
```

For example, here is the representation of the above `R` from the Roessler system using both plotting approaches:

```@example MAIN
using CairoMakie
fig = Figure(resolution = (1000,500))

ax = Axis(fig[1,1])
xs, ys = coordinates(R)
scatter!(ax, xs, ys; color = :black, markersize = 1)
ax.limits = ((1, size(R, 1)), (1, size(R, 2)));
ax.aspect = 1

ax2 = Axis(fig[1,2]; aspect = 1)
Rg = grayscale(R)
heatmap!(ax2, Rg; colormap = :grays)
fig
```

and here is exactly the same process, but using a delay embedded trajectory instead
```@example MAIN
using DelayEmbeddings

y = X[:, 2]
τ = estimate_delay(y, "mi_min")
m = embed(y, 3, τ)
E = RecurrenceMatrix(m, 5.0; metric = "euclidean")

xs, ys = coordinates(E)
fig, ax = scatter(xs, ys; markersize = 1)
ax.aspect = 1
fig
```

which justifies why recurrence plots are so fitting to be used in embedded timeseries.

!!! warning "Careful when using Recurrence Plots"
    It is easy when using `grayscale` to not change the width/height parameters. The width and height are important when in `grayscale` when the matrix size exceeds the display size! Most plotting libraries may resample arbitrarily or simply limit the displayed pixels, so one needs to be extra careful.

    Besides graphical problems there are also other potential pitfalls dealing with the conceptual understanding and use of recurrence plots. All of these are summarized in the following paper which we suggest users to take a look at:

    N. Marwan, *How to avoid potential pitfalls in recurrence plot based data analysis*, Int. J. of Bifurcations and Chaos ([arXiv](http://arxiv.org/abs/1007.2215)).

## Skeletonized Recurrence Plots

The finite size of a recurrence plot can cause border effects in the recurrence quantification-measures [`rqa`](@ref).
Also the sampling rate of the data and the chosen recurrence threshold selection method (`fixed`, `fixedrate`, `FAN`)
plays a crucial role. They can cause the thickening of diagonal lines in the recurrence matrix.
Both problems lead to biased line-based RQA-quantifiers and is discussed in:

K.H. Kraemer & N. Marwan, *Border effect corrections for diagonal line based recurrence quantification analysis measures*,
[Phys. Lett. A 2019](https://publications.pik-potsdam.de/rest/items/item_23376_6/component/file_24222/content).

```@docs
skeletonize
```

Consider, e.g. a skeletonized version of a simple sinusoidal:
```@example MAIN
using RecurrenceAnalysis, DelayEmbeddings, CairoMakie

data = sin.(2*π .* (0:400)./ 60)
Y = embed(data, 3, 15)

R = RecurrenceMatrix(Y, GlobalRecurrenceRate(0.25))
R_skel = skeletonize(R)

fig = Figure(resolution = (1000,600))
ax = Axis(fig[1,1]; title = "RP of monochromatic signal")
heatmap!(ax, grayscale(R); colormap = :grays)

ax = Axis(fig[1,2]; title = "skeletonized RP")
heatmap!(ax, grayscale(R_skel); colormap = :grays)
fig
```

This way spurious diagonal lines get removed from the recurrence matrix, which
would otherwise effect the quantification based on these lines.

## Distance matrix
The distance function used in [`RecurrenceMatrix`](@ref) and co. can be specified either as any `Metric` instance from [`Distances`](https://github.com/JuliaStats/Distances.jl). In addition, the following function returns a matrix with the cross-distances across all points in one or two trajectories:

```@docs
distancematrix
```

## `StateSpaceSet` reference
```@docs
StateSpaceSet
```