# How to Lint a Julia Project

#tech

Lint a Julia project[^gist].

```julia
using LanguageServer, StaticLint, SymbolServer

path = abspath(ARGS[1])
root_file = if length(ARGS) > 1
    abspath(ARGS[2])
else
    joinpath(path, "src", string(basename(path), ".jl"))
end

s = LanguageServerInstance(Pipe(), stdout, path)
_, symbols = SymbolServer.getstore(s.symbol_server, path)
s.global_env.symbols = symbols
s.global_env.extended_methods = SymbolServer.collect_extended_methods(s.global_env.symbols)
s.global_env.project_deps = collect(keys(s.global_env.symbols))

f = StaticLint.loadfile(s, root_file)
StaticLint.semantic_pass(LanguageServer.getroot(f))
StaticLint.check_all(LanguageServer.getcst(f), s.lint_options, LanguageServer.getenv(f, s))
empty!(f.diagnostics)
LanguageServer.mark_errors(f, f.diagnostics)
foreach(println, f.diagnostics)
```

[^gist]: https://gist.github.com/pfitzseb/22493b0214276d3b65833232aa94bf11
