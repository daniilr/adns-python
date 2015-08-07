# Introduction #

Things that should be done in the future.


# Details #

ADNS.QueryEngine is currently a wrapper around adns.init() object,
since adns-python was written prior to Python-2.2 new-style classes.
It would probably be better to make it a true subclass. (1.3.x).

~~adns-1.2 broke the API a bit: One of the private members that adns-python
depends on was renamed and made public. Unfortunately there is no easy
way to detect the adns version, so adns-python-1.2 will have a hard
dependency on adns-1.2 or newer.~~ Done 2006-12-31

~~Update memory management to use newer Python APIs. (1.3.x)~~ Done 2007-01-27 (1.2.1)