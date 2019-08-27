引子：`dependency -> tasks -> graph`

1. [initialization] determines projects
2. [configure] all project build scripts are executed，create tasks
3. [execute] execute tasks

- setting.gradle 
    - init phase
    - include project

- project location
    - file tree
    - configurable

- building the tree
    - hierarchical/flat



project tree elements can be modified



- initialization
    - finding `setting.gradle`, then determine if run multi project build
    - `settings.gradle` location
    - project objects

- configure
    - execute build scripts

- execute
    - find specificed tasks and execute

- responding to the lifecycle in the build script
     - listener or closure
    - events
      - project evaluation
      - task creation
      - task graph ready
      - task execute