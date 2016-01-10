# AttrX::InitArg

Moose like `init_arg` for Perl 6

``` perl6
use AttrX::InitArg;

class SecretEnvoy {
    has $!message is init-arg is required;
    has $.steed is init-arg(False) = 'Shadowfax';
    has $.rider is init-arg('messenger') = 'Gandalf';

    method get-message($password){
        $password eq 'opensesame' ?? $!message !! Nil;
    }

}

my $msg = SecretEnvoy.new(
    message => 'TOP SECRET',
    messenger => 'BatMan',
    steed => 'Bat-Mobile'
);

say so $msg.can('message') #-> False
say $msg.get-message('opensesame'); #-> TOP SECRET
say $msg.steed #-> Shadowfax;
say $msg.rider #-> Batman;

```

## Description

Perl 6 is presently quite opinionated about the
concepts of *public* and *private* attributes. It doesn't allow for:

1. Attributes that can be set by `.new` but do not have accessors
2. Attributes that cannot be set by `.new` but do have accessors

Which isn't to say that it isn't flexible enough to create attributes
that behave in the above way, it just involves a lot of
boilerplate. This modules takes care of that in the same way as Perl 5's
[Moose](https://metacpan.org/release/Moose). Moose has an attribute trait called
[init_arg](https://metacpan.org/pod/distribution/Moose/lib/Moose/Manual/Attributes.pod#Constructor-parameters-init_arg)
which this module attempts to emulate.

There are three ways of using init arg.

### With no argument

``` perl6
class Foo {
    has $!attr is init-arg;
    method works { say $!attr }
}
Foo.new( attr => 'win' ).works #-> win
```

This is intended to be used with `$!` attributes. Arguments matching

### With a string argument

``` perl6
class Foo {
    has $.attr is init-arg('other-name');
}
say Foo.new(other-name => "bar").attr #-> bar
say Foo.new(attr => "bar").attr #-> (Any)
```

For use with both `$!` and `$.` attributes. Sets their constructor
name. `$.` attributes will no longer be set with their usual
name in `.bless`, but it won't `die` if you try.

### With a `False` argument

```perl6
class Foo {
    has $.attr is init-arg(False) = "foo";
}

say Foo.new(attr => "bar").attr #-> foo
```

For use with `$.` attributes only. Does not allow the attribute to be
set by `.bless`. It will ignore you if you try. Usually used when you
want an attribute with accessors but always want the attribute to
start as its default value.

## Usage Notes

### .gist and .perl

This module modfiies .gist and .perl (in a fairly ugly way) so that
`.perl`'s EVALablilty is preserved. However, `.gist` is modified so
that it never prints out the values of `$!` attributes.

### Performance

Presently, this module is implemented more as an experiment. Objects
with `is init-arg` attributes will take an unnecessary performance penalty each
time the object is creadted until I've done some optimization.

### Is this even a good idea?

It depends. If you are using no-argument form of `init-arg` with `$!`
attributes, make sure you have considered just making the attribute
a `$.` attribute. Is public read-only not good enough? Does it
**have** to have no accessors?