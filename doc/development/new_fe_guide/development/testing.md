# Overview of Frontend Testing

## Types of tests in our codebase

* **RSpec**
  * **[Ruby unit tests](#ruby-unit-tests-spec-rb)** for models, controllers, helpers, etc. (`/spec/**/*.rb`)
  * **[Full feature tests](#full-feature-tests-spec-features-rb)** (`/spec/features/**/*.rb`)
* **[Karma](#karma-tests-spec-javascripts-js)** (`/spec/javascripts/**/*.js`)
* ~~Spinach~~ — These have been removed from our codebase in May 2018. (`/features/`)

## RSpec: Ruby unit tests `/spec/**/*.rb`

These tests are meant to unit test the ruby models, controllers and helpers.

### When do we write/update these tests?

Whenever we create or modify any Ruby models, controllers or helpers we add/update corresponding tests.

---

## RSpec: Full feature tests `/spec/features/**/*.rb`

Full feature tests will load a full app environment and allow us to test things like rendering DOM, interacting with links and buttons, testing the outcome of those interactions through multiple pages if necessary. These are also called end-to-end tests but should not be confused with QA end-to-end tests (`package-and-qa` manual pipeline job).

### When do we write/update these tests?

When we add a new feature, we write at least two tests covering the success and the failure scenarios.

### Relevant notes

A `:js` flag is added to the test to make sure the full environment is loaded.

```
scenario 'successfully', :js do
  sign_in(create(:admin))
end
```

The steps of each test are written using capybara methods ([documentation](http://www.rubydoc.info/gems/capybara/2.15.1)).

Bear in mind <abbr title="XMLHttpRequest">XHR</abbr> calls might require you to use `wait_for_requests` in between steps, like so:

```rspec
find('.form-control').native.send_keys(:enter)

wait_for_requests

expect(page).not_to have_selector('.card')
```

---

## Karma tests `/spec/javascripts/**/*.js`

These are the more frontend-focused, at the moment. They're **faster** than `rspec` and make for very quick testing of frontend components.

### When do we write/update these tests?

When we add/update a method/action/mutation to Vue or Vuex, we write karma tests to ensure the logic we wrote doesn't break. We should, however, refrain from writing tests that double-test Vue's internal features.

### Relevant notes

Karma tests are run against a virtual DOM.

To populate the DOM, we can use fixtures to fake the generation of HTML instead of having Rails do that.

Be sure to check the [best practices for karma tests](../../testing_guide/frontend_testing.html#best-practices).

### Vue and Vuex

Test as much as possible without double-testing Vue's internal features, as mentioned above.

Make sure to test computedProperties, mutations, actions. Run the action and test that the proper mutations are committed.

Also check these [notes on testing Vue components](../../fe_guide/vue.html#testing-vue-components).

#### Vuex Helper: `testAction`

We have a helper available to make testing actions easier, as per [official documentation](https://vuex.vuejs.org/en/testing.html):

```
testAction(
  actions.actionName, // action
  { }, // params to be passed to action
  state, // state
  [
    { type: types.MUTATION},
    { type: types.MUTATION_1, payload: {}},
  ], // mutations committed
  [
    { type: 'actionName', payload: {}},
    { type: 'actionName1', payload: {}},
  ] // actions dispatched
  done,
);
```

Check an example in [spec/javascripts/ide/stores/actions_spec.jsspec/javascripts/ide/stores/actions_spec.js](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/spec/javascripts/ide/stores/actions_spec.js).

#### Vue Helper: `mountComponent`

To make mounting a Vue component easier and more readable, we have a few helpers available in `spec/helpers/vue_mount_component_helper`.

* `createComponentWithStore`
* `mountComponentWithStore`

Examples of usage:

```
beforeEach(() => {
  vm = createComponentWithStore(Component, store);

  vm.$store.state.currentBranchId = 'master';

  vm.$mount();
},
```

```
beforeEach(() => {
  vm = mountComponentWithStore(Component, {
    el: '#dummy-element',
    store,
    props: { badge },
  });
},
```

Don't forget to clean up:

```
afterEach(() => {
  vm.$destroy();
});
```
