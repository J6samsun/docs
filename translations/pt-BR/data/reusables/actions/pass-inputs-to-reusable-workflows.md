To pass named inputs to a called workflow, use the `with` keyword in a job. Use a palavra-chave `segredos` para passar segredos nomeados. For inputs, the data type of the input value must match the type specified in the called workflow (either boolean, number, or string).

{% raw %}
```yaml
jobs:
  call-workflow-passing-data:
    uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
    with:
      username: mona
    secrets:
      envPAT: ${{ secrets.envPAT }}
```
{% endraw %}
