# Reproduce behaviour in `pants` where modifications to `__defaults__` in `BUILD` files are not detected/propagated when using `--changed-since`

Checkout the commit:
```
git checkout dd83a04f2fd46109e001033fb3ae9d88b6c9e50a
```

In this current commit, we have some `BUILD` files (`example-1/libs/BUILD` and `example-1/BUILD`) where 
`pylint`, `black` and `flake8` are skipped.

We can see that all linting tests pass:
```
pants --no-local-cache lint ::
echo $?
```
It yields (truncated):
```
✓ autoflake succeeded.
✓ bandit succeeded.
✓ isort succeeded.
0
```


## Case 1: `BUILD` file is modified

1. Modify the `example-1/libs/BUILD` file, setting all the `skip_*` args to `False` instead of `True`: 
    ```
    sed -i '' 's/True/False/g' example-1/libs/BUILD
    ```

2. Run the tests with `--changed-since`:
    ```
    pants --no-local-cache --changed-since=origin/main lint
    echo $?
    ```
    This returns:
    ```
    0
    ```

3. Check the list of modified files with `--changed-since`:
    ```
    pants --no-local-cache --changed-since=origin/main list
    echo $?
    ```
    This yields:
    ```
    16:55:35.41 [WARN] No targets were matched in goal `list`.
    0
    ```

4. Run the full linting:
    ```
    pants --no-local-cache lint ::
    echo $?
    ```
    This yields (truncated):
    ```
    (...)
    ✓ autoflake succeeded.
    ✓ bandit succeeded.
    ✕ black failed.
    ✕ flake8 failed.
    ✓ isort succeeded.
    ✕ pylint failed.

    (One or more formatters failed. Run `pants fmt` to fix.)
    16
    ```

5. Restore modified file:
    ```
    git restore example-1/libs/BUILD
    ```

## Case 2: `BUILD` file is created


1. Create a new `BUILD` file in `example-2/libs/BUILD` file, copying the parent `BUILD` and effectively setting all the `skip_*` args to `False` instead of `True`: 
    ```
    cp example-2/BUILD example-2/libs/BUILD
    sed -i '' 's/True/False/g' example-2/libs/BUILD
    git add example-2/libs/BUILD
    ```

2. Run the tests with `--changed-since`:
    ```
    pants --no-local-cache --changed-since=origin/main lint
    echo $?
    ```
    This returns:
    ```
    0
    ```

3. Check the list of modified files with `--changed-since`:
    ```
    pants --no-local-cache --changed-since=origin/main list
    echo $?
    ```
    This yields:
    ```
    17:00:53.17 [WARN] No targets were matched in goal `list`.
    0
    ```

4. Run the full linting:
    ```
    pants --no-local-cache lint ::
    echo $?
    ```
    This yields (truncated):
    ```
    (...)
    ✓ autoflake succeeded.
    ✓ bandit succeeded.
    ✕ black failed.
    ✕ flake8 failed.
    ✓ isort succeeded.
    ✕ pylint failed.

    (One or more formatters failed. Run `pants fmt` to fix.)
    16
    ```

5. Deleted the newly added file:
    ```
    git rm -f example-2/libs/BUILD
    ```
