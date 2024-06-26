+++
title = "Specs"
template = "section-nodate.html"
+++

> **S**pecs **P**arallel **ECS**

Specs is an Entity-Component System written in Rust.
Unlike most other ECS libraries out there, it provides

* easy parallelism
* high flexibility
    * contains 5 different storages for components, which can be extended by the user
    * its types are mostly not coupled, so you can easily write some part yourself and
      still use Specs
    * `System`s may read from and write to components and resources, can depend on each
      other and you can use barriers to force several stages in system execution
* high performance for real-world applications

## [Link to the book][book]

[book]: docs/tutorials/

## Example

```rust
use specs::prelude::*;

// A component contains data
// which is associated with an entity.
#[derive(Debug)]
struct Vel(f32);

impl Component for Vel {
    type Storage = VecStorage<Self>;
}

#[derive(Debug)]
struct Pos(f32);

impl Component for Pos {
    type Storage = VecStorage<Self>;
}

struct SysA;

impl<'a> System<'a> for SysA {
    // These are the resources required for execution.
    // You can also define a struct and `#[derive(SystemData)]`,
    // see the `full` example.
    type SystemData = (WriteStorage<'a, Pos>, ReadStorage<'a, Vel>);

    fn run(&mut self, (mut pos, vel): Self::SystemData) {
        // The `.join()` combines multiple component storages,
        // so we get access to all entities which have
        // both a position and a velocity.
        for (pos, vel) in (&mut pos, &vel).join() {
            pos.0 += vel.0;
        }
    }
}

fn main() {
    // The `World` is our
    // container for components
    // and other resources.
    let mut world = World::new();
    world.register::<Pos>();
    world.register::<Vel>();

    // An entity may or may not contain some component.

    world.create_entity().with(Vel(2.0)).with(Pos(0.0)).build();
    world.create_entity().with(Vel(4.0)).with(Pos(1.6)).build();
    world.create_entity().with(Vel(1.5)).with(Pos(5.4)).build();

    // This entity does not have `Vel`, so it won't be dispatched.
    world.create_entity().with(Pos(2.0)).build();

    // This builds a dispatcher.
    // The third parameter of `with` specifies
    // logical dependencies on other systems.
    // Since we only have one, we don't depend on anything.
    // See the `full` example for dependencies.
    let mut dispatcher = DispatcherBuilder::new()
        .with(SysA, "sys_a", &[]).build();
    // This will call the `setup` function of every system.
    // In this example this has no effect
    // since we already registered our components.
    dispatcher.setup(&mut world);

    // This dispatches all the systems in parallel (but blocking).
    dispatcher.dispatch(&mut world);
}
```

## Contribution

Contribution is very welcome! If you didn't contribute before,
just filter for issues with "easy" or "good first issue" label.
Please note that your contributions are assumed to be dual-licensed under Apache-2.0/MIT.