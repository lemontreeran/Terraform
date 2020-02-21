### Transforming lists and maps<sup>2</sup>

#### List comprehensions in Python

Terraform `for` expressions are grammatically similar to and actually modelled on the list comprehension feature of Python and Haskell, which was specifically requested in the original [feature request](https://github.com/hashicorp/terraform/issues/8439). And, for anyone unfamiliar with Python's list comprehension, it is similar to the map feature of other languages like Ruby and Perl 5.

A list comprehension in Python looks like this:

```python
>>> numbers = [1, 2, 3, 4]
>>> squares = [n**2 for n in numbers]
>>> squares
[1, 4, 9, 16]
```

And they can be further filtered by adding an `if` expression, like this:

```python
>>> even = [n for n in squares if n % 2 == 0]
>>> even
[4, 16]
```

#### For expressions

Terraform's `for` expression, meanwhile, is pretty much the same. A [`for` expression](https://www.terraform.io/docs/configuration/expressions.html):

> ... creates a complex type value by transforming another complex type value. Each element in the input value can correspond to either one or zero values in the result, and an arbitrary expression can be used to transform each input element into an output element.

#### Example 6: Transforming a list into another list

Here is an example:

```js
locals {
  arr = ["host1", "host2", "host3"]
}

output "test" {
  value = [for s in local.arr: upper(s)]
}
```

Applying that:

```text
▶ terraform012 apply

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

test = [
  "HOST1",
  "HOST2",
  "HOST3",
]
```

And for comparison, the same code in Python would be:

```python
>>> arr = ["host1", "host2", "host3"]
>>> value = [s.upper() for s in arr]
>>> value
['HOST1', 'HOST2', 'HOST3']
```

Very pythonic.

#### Example 7: Filtering a list

And again, as in Python, Terraform lists can also be filtered by adding an `if` in the `for` expression:

```js
locals {
  arr = [1,2,3,4,5,6,7,8,9,10]
}

output "test" {
  value = [for n in local.arr: n if n > 5]
}
```

And for reference, in Python, that would be:

```python
>>> arr = [1,2,3,4,5,6,7,8,9,10]
>>> value = [n for n in arr if n > 5]
>>> value
[6, 7, 8, 9, 10]
```

#### Example 8: Transforming a list into a map

Map transformations in Terraform are also Pythonic. If I change my Terraform code to:

```js
locals {
  arr = ["host1", "host2", "host3"]
}

output "test" {
  value = {for s in local.arr : s => upper(s)}
}
```

And apply that, I get:

```text
▶ terraform012 apply

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

test = {
  "host1" = "HOST1"
  "host2" = "HOST2"
  "host3" = "HOST3"
}
```

And again comparing this with Python:

```python
>>> arr = ["host1", "host2", "host3"]
>>> value = {s: s.upper() for s in arr}
>>> value
{'host3': 'HOST3', 'host2': 'HOST2', 'host1': 'HOST1'}
```

#### Example 9: Transforming a map into a list

To iterate over a map, Terraform provides a `keys()` and `values()` function, similar to corresponding methods in Python. Thus, this sort of thing is possible:

```js
locals {
  mymap = {
    "foo" = { "id" = 1 }
    "bar" = { "id" = 2 }
    "baz" = { "id" = 3 }
  }
}

output "test" {
  value = [for v in values(local.mymap): v["id"]]
}
```

Which is to Terraform as this is to Python:

```python
>>> mymap = {'foo': {'id': 1}, 'bar': {'id': 2}, 'baz': {'id': 3}}
>>> value = [v['id'] for v in mymap.values()]
>>> value
[1, 2, 3]
```

#### Example 10: A real life example

All of this is good and theoretical although the reader may want a real life example at this point. Here is one:

```js
variable "vpc_id" {
  description = "ID for the AWS VPC where a security group is to be created."
}

variable "subnet_numbers" {
  description = "List of 8-bit numbers of subnets of base_cidr_block that should be granted access."
  default = [1, 2, 3]
}

data "aws_vpc" "example" {
  id = var.vpc_id
}

resource "aws_security_group" "example" {
  name        = "friendly_subnets"
  description = "Allows access from friendly subnets"
  vpc_id      = var.vpc_id

  ingress {
    from_port = 0
    to_port   = 0
    protocol  = -1

    cidr_blocks = [
      for num in var.subnet_numbers:
      cidrsubnet(data.aws_vpc.example.cidr_block, 8, num)
    ]
  }
}
```

So, for each subnet number, use the `cidrsubnet()` function to generate a corresponding CIDR. The VPC's CIDR prefix is 10.1.0.0/16, it would yield `["10.1.1.0/24", "10.1.2.0/24", "10.1.3.0/24"]`.

Since this was not really possible in Terraform 0.11 and earlier without a lot of violating DRY, it can immediately be seen that the `for` expressions are a big leap forward for the Terraform community.
