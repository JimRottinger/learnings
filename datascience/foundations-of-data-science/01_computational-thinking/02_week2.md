# Notes on Week 2

## Exponential Growth

We often want to acquire the growth rate over some period of time. Say that we have a two numbers for the federal budget of the United States. In 2002, it was 2 trillion and in 2012, it was 3 trillion. What is the growth rate per year? The equation is:

```python
G = (after/before)**(1/t) - 1
```

With t being 10 \(number of years\), that gives us a growth rate of 4.138%. Then, what if we want to know how big the budget will be in 2018 if the growth rate stays the same?

```python
after = before * (1+G)**t
```

In our case, before is the value in 2012 and t is \(2018-2012\), or 6 years. This gives as a 2018 budget of 3.82 trillion. You can also plug in the number for 2002 and use a t of 16 and you will get the exact same number.

