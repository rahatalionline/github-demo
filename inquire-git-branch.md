## Inquire Git Branch Details
This example explains how to add an additional button to the application footer which when clicked will display details of the cuurent Git branch. The additional button will be displayed with a GitHub icon in the standard SugarCRM footer with 'Inquire' as its label. When user clicks on it, it will display name of the current Git branch in a span placed next to the Inquire button and as well as show an information alert displaying following details about the current Git branch:

- Branch Name
- Committer date
- Branch Hash

![Footer Before](supporting_images/Inquire_Git_Branch_Details_Before Click.png)
![Footer After](supporting_images/Inquire_Git_Branch_Details_After_Click.png)
![Alert](supporting_images/Inquire_Git_Branch_Details_Alert.png)

For this example, we will create a custom view and then append the view component to the footer layout metadata. This tutorial requires the following steps, which are explained in the sections below:
1. Creating the Custom View
1. Appending the View to Layout Metadata
1. Adding Custom labels to the Extensions Framework
1. Creating a Custom Api for getting current Git branch Details

### Creating the Custom View
To add a button to the application footer, you will first need to create the custom view that will contain your button and logic. To create the custom view create a folder in **./custom/clients/base/views** with the name of your view, such as **./custom/clients/base/views/inquire-git-branch/**. Once your folder is in place, you can create the three primary files that make up your view:

```
1. inquire-git-branch.hbs  - Contains the handlebar template for your view
2. inquire-git-branch.js   - Contains the JavaScript controller logic
3. inquire-git-branch.php  - Contains the metadata your view might use. For this example,
                             we simply use metadata to define whether greeting will auto close or not.
```

Now let's create all of these three files one by one.

**./custom/clients/base/views/inquire-git-branch/inquire-git-branch.hbs**
```javascript
{{!
    Define HTML for our new button. We will mimic the style of other buttons
    in the footer so we remain consistent.
    Also we will only show the button if a User is logged into the system.
}}

{{#if isAuthenticated}}
<button class="btn btn-invisible" type="button" data-action="inquire-git-branch">
    <i class="fa fa-github"></i>
    <span class="action-label"> {{str "INQUIRE_GIT_BRANCH"}}</span>
    <span id="git-branch-name" class="action-label"></span>
</button>
{{/if}}
```

**./custom/clients/base/views/inquire-git-branch/inquire-git-branch.js**
```javascript
({
    // tagName attribute is inherited from Backbone.js.
    // We set it to "span" instead of default "div" so that our "button" element is displayed inline.
    tagName: "span",
    events: {
        //On click of our "button" element
        'click [data-action=inquire-git-branch]': 'inquireGitBranch'
    },
    isAuthenticated: false,

    _renderHtml: function() {
        this.isAuthenticated = app.api.isAuthenticated();
        this._super('_renderHtml');
    },
    inquireGitBranch: function(){
        var self = this;

        // Displaying Processing alert to the user
        app.alert.show('inquiring-git-branch-process', {
            level: 'process',
            title: app.lang.get('INQUIRING_GIT_BRANCH_DETAILS')
        });

        app.api.call('GET', app.api.buildURL('GetGitBranchDetails'), null, {},
            {
                success: function (branch) {
                    if (_.isEmpty(branch)) {
                        return;
                    }

                    // Display current Git branch name in footer
                    $("#git-branch-name").html(": " + branch.name);

                    var details =   '<table><tr><td><i class="fa fa-code-fork"></i></td><td>'+branch.name+'</td></tr>'+
                                    '<tr><td><i class="fa fa-calendar"></i></td><td>'+branch.date+'</td></tr>'+
                                    '<tr><td><i class="fa fa-barcode"></i></td><td>'+branch.hash+'</td></tr></table>';

                    app.alert.show('inquiring-git-branch-info', {
                        level: 'info',
                        title: 'Current Git Branch Details:',
                        messages: details
                    });
                },
                error: function(error) {
                    app.alert.show('inquiring-git-branch-error', {
                        level: 'error',
                        messages: 'Unable to Retrieve Git Details!'
                    });
                },
                complete: function() {
                    app.alert.dismiss('inquiring-git-branch-process');
                }

            }
        );
    }
})
```

**./custom/clients/base/views/inquire-git-branch/inquire-git-branch.php**
```php
<?php

$viewdefs['base']['view']['inquire-git-branch'] = array(
    'autoClose' => false
);
```

### Appending the View to Layout Metadata
Once the view is created, you simply need to add the view to the Footer Layout metadata. The Footer Layout is located in **./clients/base/layotus/footer/**, so appending the view to this layout can be done via the Extension Framework. Create a file in **./custom/Extension/application/Ext/clients/base/layouts/footer/** and the Extension Framework will append the metadata to the Footer Layout.

**./custom/Extension/application/Ext/clients/base/layouts/footer/inquire-git-branch.php**
```php
<?php

$viewdefs['base']['layout']['footer']['components'][] = array (
    'view' => 'inquire-git-branch',
);
```

### Adding Custom labels to the Extensions Framework
In this section we create a custom language file in the Extension Framewrok which will contains the labels we are using to display our custom button and the process alert which gets displayed when the user clicks on the Inquire button.

**./custom/Extension/application/Ext/Language/en_us.inquire-git-branch.php**
```php
<?php

$app_strings['INQUIRE_GIT_BRANCH'] = 'Inquire';
$app_strings['INQUIRING_GIT_BRANCH_DETAILS'] = 'Inquiring Git branch details';
```

### Creating a Custom Api for getting current Git branch Details
Finally we create a custom Api for getting the current branch details from the file system using PHP exec function. This Api grabs curent branch name, committer date and current branch hash and return it to the caller as an array.

**./custom/clients/base/api/GitApi.php**
```php
<?php

if(!defined('sugarEntry') || !sugarEntry) die('Not A Valid Entry Point');

class GitApi extends SugarApi
{
    public function registerApiRest()
    {
        return array(
            //GET
            'GitEndpoint' => array(
                //request type
                'reqType' => 'GET',

                //set authentication
                'noLoginRequired' => false,

                //endpoint path
                'path' => array('GetGitBranchDetails'),

                //endpoint variables
                'pathVars' => array(''),

                //method to call
                'method' => 'GrabGitBranchDetails',

                //short help string to be displayed in the help documentation
                'shortHelp' => 'Get details of the current Git branch'
            ),
        );
    }

    // Get Git branch details & return it to the caller
    public function GrabGitBranchDetails($api, $args)
    {
        // Branch Name
        exec('git symbolic-ref HEAD', $name);
        $name = end(explode('/', $name[0]));

        // Committer Date
        exec('git show -s --format=%cd', $date);

        // Branch Hash
        exec('git rev-parse HEAD', $hash);

        $aGitBranchDetials = array(
            'name' => $name,
            'date' => $date,
            'hash' => $hash
        );

        return $aGitBranchDetials;
    }
}
?>
```

Once all the files are in place, you can run a Quick Repair and Rebuild and the new button will show in the footer when logged into your system.
