It sounds certainly obvious but it's not. 

Whenever you're making a mod for anything you automatically become a researcher, at least temporarily, and to be a researcher in IT you will need the forbidden knowledge that is "google search".

Most people do not really know how to google search and therefore resort to youtube videos or asking other people even when the resources are widely available to solve your issue yourself.

# Step 1: Figure out the issue

It's not always obvious what the issue is at all times. Let's take this example:

```
# pip3 install pycparser

Collecting pycparser
  Using cached https://files.pythonhosted.org/packages/68/9e/49196946aee219aead1290e00d1e7fdeab8567783e83e1b9ab5585e6206a/pycparser-2.19.tar.gz
Building wheels for collected packages: pycparser
  Running setup.py bdist_wheel for pycparser ... error
  Complete output from command /usr/bin/python3 -u -c "import setuptools, tokenize;__file__='/tmp/pip-install-g_v28hpp/pycparser/setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" bdist_wheel -d /tmp/pip-wheel-__w_f6p0 --python-tag cp36:
  Traceback (most recent call last):
    File "<string>", line 1, in <module>
    ...
    File "/usr/lib/python3.6/site-packages/pkg_resources/__init__.py", line 2349, in resolve
      module = __import__(self.module_name, fromlist=['__name__'], level=0)
  ModuleNotFoundError: No module named 'wheel.bdist_wheel'

  ----------------------------------------
  Failed building wheel for pycparser
  Running setup.py clean for pycparser
Failed to build pycparser
Installing collected packages: pycparser
  Running setup.py install for pycparser ... done
Successfully installed pycparser-2.19
```

What is the issue here?

Clearly something's wrong with pip and/or the python environment but the errors might be confusing so you always need to read from the bottom up and see if there's any keywords that could help you.

In this case the keywords seem to be "module" "wheel" "pycparser", if you managed to find them you're already pretty good.


# Step 2: Looking it up

How do we look this up on google?

If your answer was something like putting

```
Building wheels for collected packages: pycparser
  Running setup.py bdist_wheel for pycparser ... error
  Complete output from command /usr/bin/python3 -u -c "import setuptools, tokenize;__file__='/tmp/pip-install-g_v28hpp/pycparser/setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" bdist_wheel -d /tmp/pip-wheel-__w_f6p0 --python-tag cp36:
  Traceback (most recent call last):
    File "<string>", line 1, in <module>
```
in the search bar you're completely wrong and will fail 99% of the times.

Queries on google need to be short, and highlight keywords, but how do we do that?

When searching on google you can use quotation marks " to signal to google that the specific word MUST exist in the page. Let's make an example:

Bad: Failed building wheel for pycparser
Good: "wheel" "pycparser" failed

Why is that? Because failed is a variable word, it could be fail, failure, unsuccessful in the end result, but pycparser will never be called pycparsemaybe, it's a constant keyword therefore must be included as it is in the google query.

# Step 3: Keep looking

tep 3: Keep looking

If you found literally nothing it means one of two things:
1. Bad keyword selection
2. First ever error like that in the entire world

You can do your guessing and figure out which one is more probable given any specific issue that might arise on your machine.

# Step 4: When in doubt start over

If you feel like you're truly stuck start from scratch and try to solve one small issue at a time, it will help a lot more than you think
