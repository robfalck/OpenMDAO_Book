# How OpenMDAO Represents Variables

In general, a numerical model can be complex, multidisciplinary, and heterogeneous.
It can be decomposed into a series of smaller computations that are chained together by passing variables from one to the next.

In OpenMDAO, we perform all these numerical calculations inside a **Component**, which represents the
smallest unit of computational work the framework understands. Each component will output its own set of variables.
Depending on which type of calculation you're trying to represent, OpenMDAO provides different kinds of components
for you to work with.

## A Simple Numerical Model

In order to understand the different kinds of components in OpenMDAO,
let us consider the following numerical model that takes $x$ as an input:

\begin{array}{l l}
    y \quad \text{is computed by solving:} &
    \cos(x \cdot y) - z \cdot y = 0  \\
    z \quad \text{is computed by evaluating:} &
    z = \sin(y) .
\end{array}

## The Three Types of Components

In our numerical model, we have three variables: $x$, $y$, and $z$.
Each of these variables needs to be defined as the output of a component.
There are three basic types of components in OpenMDAO:

1. **IndepVarComp** : defines independent variables (e.g., x)
2. **ExplicitComponent**: defines dependent variables that are computed explicitly (e.g., z)
3. **ImplicitComponent**`: defines dependent variables that are computed implicitly (e.g., y)

The most straightforward way to implement the numerical model would be to assign each variable its own component, as below.

 | No. | Component type    | Inputs  | Outputs |
 |-----|-------------------|---------|---------|
 | 1   | IndepVarComp      |         |   x     |
 | 2   | ImplicitComponent |  x, z   |   y     |
 | 3   | ExplicitComponent |   y     |   z     |

Another way that is also valid would be to have one component compute both y and z explicitly,
which would mean that this component solves the implicit equation for y internally.

| No.  | Component type    | Inputs | Outputs |
|------|-------------------|--------|---------|
|   1  | IndepVarComp      |        |    x    |
|   2  | ExplicitComponent |    x   |   y, z  |
|------|-------------------|--------|---------|

Both ways would be valid, but the first way is recommended.
The second way requires the user to solve y and z together, and computing the derivatives of y and z with respect to x is non-trivial.
The first way would also require implicitly solving for y, but an OpenMDAO solver could converge that for you.
Moreover, for the first way, OpenMDAO would automatically combine and assemble the derivatives from components 2 and 3.
