# Role Based Authorization

_user_ `users plays a certain role`
_John => moderator, Sally => manager, Frank => owner_ `roles comes with certain ability`
_moderator => 'edit_forum'_
_owner => 'view_financialreports_

_Create a migration_ `php artisan make:migration create_roles_tables --table=roles`

_NB: Roles include certain abilities_

`roles table data`
_role id_
_role name_
_role label_
_role timestamps_

`abilities table data` on the same migration file
_same as above_
_ability name as edit_forum_
_ability label as Edit the Form_

the two above is a _Many to Many Relationship_

`ability_role` table since we have a _Many to Many Relationship_
_primary(['role_id', 'ability_id'])_

_Run php artisan migrate_
_php artisan make:mode Role_ A Role that the user plays
_php artisan make:mode Ability_ Associated with that role

-Role.php
_A Role belongsToMany abilities_

-Ability.php
_Ability belongsToMany roles_ Roles that include that ability \$moderator->abilities `Role that has certain ability associated to them

-User.php
_\$user->roles_ grab all roles assign to a user

_$user->roles()->save($role);_ assign a role to a user `$user->assignRole()`

_assign a new ability to a role_

_\$user->roles;_ display all roles assign to the currrent user (collection of roles)

_\$user->roles[0]->abilities;_ display abilities for the target role

_\$user->roles->map->abilities;_ `map over each role and return abilities associated with that role` return collection of collection for each role

_\$user->roles->map->abilities->flatten();_ `flatten down to a single collection`

_\$user->roles->map->abilities->flatten()->pluck('name');_ pluck out the name

_\$user->roles->map->abilities->flatten()->pluck('name')->unique();_ grab the unique ones

_\$user->abilities();_ user give their abilities

-blade view

---

@can('edit_forum')

<li>
<a href="#">Edit Forum</a>
</li>
@endcan

---

-AuthServiceProvider to define a gate for the _edit_forum_ ability in the boot function

-Global before hook or filter
_Run this `Gate::before(function($user, $ability){return $user->abilities()->contains($ability);});`_ before any authorization at the moment the function is trigger, we then have an authenticated user, we now check if he contains the current ability in the list of abilities

_In the web.php_ file simulates an authenticated user by adding this line `auth()->loginUsingId(13);`

_User.php_ assignRole(\$role) function `$this->roles()->sync($role, false);` Add any new record if necessary to avoid duplication of assigning the same role to a user

_Role.php_ allowTo function `return $this->abilities()->sync($ability, false);` Avoid duplications

-blade view

---

@can('view_reports')

<li>
<a href="/reports">View Report</a>
</li>
@endcan

---

-web.php

---

Route::get('/reports', function () {
return 'the secret reports';
})->middleware('can:view_reports');

---

-User.php

---

public function assignRole($role)
{
if (is_string($role)) {
$role = Role::whereName($role)->firstOrFail();
}

$this->roles()->sync($role, false);
}

---

To be able to run `$user->assignRole('manager');` on tinker, make sure you add the following code to the _assignRole(\$role)_ function in the _User.php_ file

---

if (is_string($role)) {
$role = Role::whereName(\$role)->firstOrFail();
}

---

Do the same thing on the Role Model

---

if (is_string($ability)) {
$ability = Ability::whereName(\$ability)->firstOrFail();
}

---
