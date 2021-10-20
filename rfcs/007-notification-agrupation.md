- Start Date: 2021-10-14
- RFC PR: (leave this empty)

# Summary

Some times many users can interact with a single entity resulting in alot of notifications, we're attempting to solve this by creating a notification groping functionality that allows Kanvas to group notification if there are various notifications in a given period of time.

This solution will cotain:
- A flag to turn on and off this feature
- A soft cap and a hard cap to set the time of the agrupation
- A grouping mechanism based on the soft and hardcap

# Example

- We're gonna add 2 properties:

```php
    /**
     * The minimum time to consider before grouping notifications.
     *
     * @var bool
     */
    protected int $softCap = 1;

    /**
     * The maximun time to consider before grouping notifications.
     *
     * @var bool
     */
    protected int $hardCap = 5;

    /**
     * Set the soft cap.
     *
     * @param int $softCap
     *
     * @return void
     */
    public function setSoftCap(int $softCap) : void
    {
        $this->softCap = $softCap;
    }

    /**
     * Set the hard cap.
     *
     * @param int $hardCap
     *
     * @return void
     */
    public function setHardCap(int $hardCap) : void
    {
        $this->hardCap = $hardCap;
    }
```

- We're gonna add a private method to determinates if the notification need to be a group:

```php
    /**
     * Identify if notifcationes should be a group.
     *
     * @return bool
     */
    private function isGroupable() : ?int
    {
        $notificationId = null;
        $notification = Notifications::findFirst([
            'conditions' => '\notification_type_id = :notificationTypeId: AND entity_id = :entityId:',
            'bind' => [
                'notificationTypeId' => $this->type->getId(),
                'entityId' => $this->entity->getId()
            ],
            'order' => 'updated_at DESC'
        ]);

        $startingTime = Carbon::now();
        $notificationTime = Carbon::parse($notification->updated_at);

        $diffTime = $notificationTime->diffInMinutes($startingTime);

        if($diffTime >= $this->softCap && $diffTime <= $this->hardCap){
            $notificationId = $notification->getId();
        }

        return $notificationId;
    }
```

- We're gonna add a private agrupation method:

```php

    /**
     * Groups a set of notifications.
     *
     * @return void
     */
    private function groupNotification() : void
    {
        $notificationGroup = json_decode($this->currentNotification->group);
        $notificationGroup->from_users[] = $this->fromUser->getId();
        $this->currentNotification->group = json_encode($notificationGroup);
    }
```

- Finally we're gonna modify the `saveNotification` method:

```php
    /**
     * Save the notification used to the database.
     *
     * @return bool
     */
    public function saveNotification() : bool
    {
        $content = $this->message();
        $app = Di::getDefault()->get('app');

        //verify if need to be a group
        $isGroupable = $this->isGroupable();

        //save to DB
        if(is_null($isGroupable)){
            $this->currentNotification = new Notifications();
            $this->currentNotification->from_users_id = $this->fromUser->getId();
            $this->currentNotification->users_id = $this->toUser->getId();
            $this->currentNotification->companies_id = $this->fromUser->currentCompanyId();
            $this->currentNotification->apps_id = $app->getId();
            $this->currentNotification->system_modules_id = $this->type->system_modules_id;
            $this->currentNotification->notification_type_id = $this->type->getId();
            $this->currentNotification->entity_id = $this->entity->getId();
            $this->currentNotification->content = $content;
            $this->currentNotification->read = 0;
            $this->currentNotification->saveOrFail();
        } else {
            $this->currentNotification = Notifications::findFirstById($isGroupable);
            $this->groupNotification();
            $this->currentNotification->saveOrFail();

        }

        return true;
    }
```



# Motivation

Enchance the way end users use notifications.

# Detailed design

Describe the proposal in details:

- Explaining the design so that someone who knows Kanvas can understand and someone who works on it can implement the proposal. 
- Think about edge-cases and include examples.

# Tradeoffs

# Alternatives

Moving this responsability to the client part

# Unresolved questions