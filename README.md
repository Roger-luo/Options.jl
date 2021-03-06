# Configurations

[![CI](https://github.com/Roger-luo/Configurations.jl/workflows/CI/badge.svg)](https://github.com/Roger-luo/Configurations.jl/actions)
[![codecov](https://codecov.io/gh/Roger-luo/Configurations.jl/branch/master/graph/badge.svg?token=U604BQGRV1)](https://codecov.io/gh/Roger-luo/Configurations.jl)
[![][docs-dev-img]][docs-dev-url]

Configurations & Options made easy.

## Installation

<p>
Configurations is a &nbsp;
    <a href="https://julialang.org">
        <img src="https://raw.githubusercontent.com/JuliaLang/julia-logo-graphics/master/images/julia.ico" width="16em">
        Julia Language
    </a>
    &nbsp; package. To install Configurations,
    please <a href="https://docs.julialang.org/en/v1/manual/getting-started/">open
    Julia's interactive session (known as REPL)</a> and press <kbd>]</kbd> key in the REPL to use the package mode, then type the following command
</p>

```julia
pkg> add Configurations
```

## Usage

This package provides a macro `@option` to let you define `struct`s to represent options/configurations, and serialize between
different option/configuration file formats such as `TOML`.

You can easily create hierarchical struct types:

```julia
julia> using Configurations

julia> "Option A"
       @option "option_a" struct OptionA
           name::String
           int::Int = 1
       end

julia> "Option B"
       @option "option_b" struct OptionB
           opt::OptionA = OptionA(;name = "Sam")
           float::Float64 = 0.3
       end
```

and then convert from a `Dict` to your option type via [`from_dict`](@ref):

```julia
julia> d = Dict{String, Any}(
           "opt" => Dict{String, Any}(
               "name" => "Roger",
               "int" => 2,
           ),
           "float" => 0.33
       );

julia> option = from_dict(OptionB, d)
OptionB(;
    opt = OptionA(;
        name = "Roger",
        int = 2,
    ),
    float = 0.33,
)
```

When there are multiple possible option types for one field,
one can use the alias to distinguish them

```julia
julia> @option struct OptionD
           opt::Union{OptionA, OptionB}
       end

julia> d1 = Dict{String, Any}(
               "opt" => Dict{String, Any}(
                   "option_b" => d
               )
           );

julia> from_dict(OptionD, d1)
OptionD(;
    opt = OptionB(;
        opt = OptionA(;
            name = "Roger",
            int = 2,
        ),
        float = 0.33,
    ),
)
```

Or you can create it from keyword arguments, e.g

```julia
julia> from_kwargs(OptionB; opt_name="Roger", opt_int=2, float=0.33)
OptionB(;
    opt = OptionA(;
        name = "Roger",
        int = 2,
    ),
    float = 0.33,
)
```

For option types you can always convert an `AbstractDict` to a given option type,
or convert them back to dictionary via `to_dict`, e.g

```julia
julia> Configurations.to_dict(option)
OrderedDict{String, Any} with 2 entries:
  "opt"   => OrderedDict{String, Any}("name"=>"Roger", "int"=>2)
  "float" => 0.33
```

For serialization, you can use the builtin TOML support

```julia
julia> to_toml(option)
"float = 0.33\n\n[opt]\nname = \"Roger\"\nint = 2\n"
```

or serialize it to other formats from `OrderedDict`.

## License

MIT License

[docs-dev-img]: https://img.shields.io/badge/docs-dev-blue.svg
[docs-dev-url]: https://rogerluo.me/Configurations.jl/dev/
