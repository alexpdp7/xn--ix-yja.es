# Infrastructure

To bootstrap, ensure you can `ssh root@ñix.es`, and then:

```
uv run ansible-playbook -i ssh-root.yaml -i production.yaml site.yaml
```
