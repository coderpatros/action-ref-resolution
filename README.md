# GitHub Action Ref Resolution Abuse

A collection of my observations while researching how GitHub Action ref
resolution _actually_ works and how it can be abused.

## Tag Branch Confusion

This one I was a little surprised by as it is directly in contradiction of the
expected documented behaviour here [https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_iduses](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_iduses).

The documentation states

```In the first option, {ref} can be a SHA, a release tag, or a branch name. If a release tag and a branch have the same name, the release tag takes precedence over the branch name. Using the commit SHA is the safest option for stability and security. For more information, see Secure use reference.```

Interestingly though, I have observed that a matching branch name takes
precedence over a release tag. Although they may fix this behaviour in the 
future.

This would allow a contributor to very sneakily create a branch named `v1` that 
would take over any GitHub action references that used `v1`.

## Shadow Commit Resolution

In this attack we leverage the underlying mechanism that GitHub uses to manage
forked repos at scale. Namely, the underlying git objects on disk are shared.

This has some advantages, it makes creating a fork, even of a very large repo,
very quick. It obviously also saves a lot on storage space. And before you go
down the rabbit hole of hash collisions possibilities, they have _alledgedly_
have additional controls in place for when that becomes a viable attack scenario. 
Although we'll have to test that out in the future.

This repo [https://github.com/coderpatros/donotuse-setup-dotnet](https://github.com/coderpatros/donotuse-setup-dotnet) 
is a fork of the official upstream `setup-dotnet` GitHub action.

Can you figure out which reference is an official release and which one is in 
our "malicious" fork?

`actions/setup-dotnet@c2fa09f4bde5ebb9d1777cf28262a3eb3db3ced7`

or

`actions/setup-dotnet@a55eb9ae83b21bfc3efab1938edeffe211c9fb03`

## Unicode for all my friends!

The universal way to abuse invisible characters and trick humans.

The recommended way of securely referencing GitHub actions is with the full
commit hash. GitHub prevents branch names that are 40 hex characters so we 
can't abuse commit hash branch resolution confusion. But a nice little 
zero width space character is fine.

These are not the same

`coderpatros/action-hello-world@6aa22f48b007348666f676eabda20e9884ed1a7f`

`coderpatros/action-hello-world@6aa22f48b007348666f676eabda20e9884ed1a7f​`
