# Application Test Role

## Purpose

The `application_test` role prevents an Employee Directory revision from
being deployed unless its pytest suite passes.

## Execution location

The role executes on the Ansible control node using:

````yaml
delegate_to: localhost


Managed servers are not changed before the test gate passes.

## Test process

1. Create a temporary workspace.
2. Clone the requested Git revision.
3. Create an isolated Python virtual environment.
4. Install test dependencies.
5. Run pytest.
6. Remove the temporary workspace.

Cleanup runs whether the test suite passes or fails.

## Revision selection

The role tests:

```yaml
application_test_version: "{{ employee_app_version }}"
````

This ensures that Ansible tests the same revision it intends to deploy.

## Failure behaviour

A non-zero pytest exit code stops the play.

No managed-host deployment roles run after a failed test.

# Application Test Role

## Purpose

The `application_test` role prevents an Employee Directory revision from
being deployed unless its pytest suite passes.

## Execution location

The role executes on the Ansible control node using:

```yaml
delegate_to: localhost
```
