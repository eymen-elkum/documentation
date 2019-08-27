# View


View is a most often used class in frontend. Every page is rendered by multiple view objects. Views can have child views. Childs views can have their own child views. And so on. When a parent view is rendered (by calling render() method), it generates HTML from all children and add it to DOM.


View file `client/custom/src/views/test/my-custom-view.js`:

```js
define('custom:views/test/my-custom-view', 'view', function (Dep) {

    return Dep.extend({

        // template file, contents is printed below
        template: 'custom:test/my-custom-view',

        // optionaly template can be defined righ here
        //templateContent: '<div class="some-test-container">{{{someKeyName}}}</div>',

        // custom initializing logic
        setup: function () {
            // calling parent setup method, you can omit it
            Dep.prototype.setup.call(this);

            // when we create a child view in setup method, rendering of the view will be held off
            // until the child view is loaded (ready)
            this.createView('someKeyName', 'custom:test/my-custom-child-view', {
                el: this.getSelector() + ' .some-test-container',
                someParam: 'test',
            });

            this.on('remove', function () {
                // is called when the view is removed
                // can be usefuly to destroy some event listeners inialized for the view
            }, this);
        },

        // is called after contents is added to DOM
        afterRender: function () {
            console.log(this.$el); // view container DOM element

            this.getView('someKeyName') // get child view
            this.clearView('someKeyName') // destroy child view
        },
    });
});
```

Template file `client/custom/res/templates/test/my-custom-view.tpl`:

```
<div class="some-test-container">{{{someKeyName}}}</div>

```

## Waiting for some data loaded before rendering

Sometimes we need to get some data loaded asynchronously before the view is rendered. For this purpose we can use `wait` method inside `setup` method. 

Wait method can receive a promise:

```js
    setup: function () {
        this.wait(
            Promise.all([
                this.model.fetch(),
                this.model.collection.fetch()
            ])
        );
    },
````

Model factory returs a promise.

```js
    setup: function () {
        this.wait(
            this.getModelFactory().create('Case')
            .then(
                function (model) {
                    model.id = this.model.id
                    return model.fetch();
                }.bind(this)
            )
            .then(
                function (model) {
                    console.log(model);
                }
            )
        );
    },
```

Wait for model is fetched. Fetch returns a promise.

```js
    setup: function () {
        this.wait(
            this.model.fetch()
        );
    },

```

A simple way to wait:

```js
    setup: function () {
        this.wait(true); // this holds off the rendering

        Espo.Ajax.getRequest('Some/Request').then(function (response) {
            this.wait(false); // this cancels waiting and proceeds to rendering
        }.bind(this));
    },

```