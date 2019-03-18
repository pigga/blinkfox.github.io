---
title: OpenStack 两种虚拟机迁移方式
date: 2019-03-18 18:40:00 +0800
layout: post
permalink: /blog/2019/03/18/OpenStack_instance_migrate.html
categories:
  - OpenStack
tags:
  - OpenStack
  - VM migrate
  - 云计算
---

###### OpenStack nova compute supports two flavors of Virtual Machine (VM) migration:


- Cold migration(冷迁移) -- migration of a VM which requires the VM to be powered off during the migrate operation during which time the VM is inaccessible.
- Hot or live migration(热迁移) -- zero down-time migration whereupon the VM is not powered off during the migration and thus remains accessible.

Understanding these VM migration operations from an OpenStack internals perspective can be a daunting task. I had the pleasure of digging into these flows in the latter part of 2013 and as part of that effort created a rough outline of the internal flows. Other's I've worked with found these flow outlines useful and thus they're provided below.<br/>


Note -- The outlines below were created based on the OpenStack source in late 2013 and thus reflect the state of OpenStack at that point in time.<br/>





**Live Migration Flow:**
```
nova.api.openstack.compute.contrib.admin_actions._migrate_live()
nova.compute.api.live_migrate()
    update instance state to MIGRATING state
    call into scheduler to live migrate (scheduler hint will be set to the host select (which may be none)).
nova.scheduler.manager.live_migration()
nova.scheduler.manager._schedule_live_migration()
nova.conductor.tasks.live_migrate.LiveMigrationTask.execute()
    check that the instance is running
    check that the instance's host is up
    if destination host provided, check that it..
        is different than the instance's host
        is up
        has enough memory
        is compatible with the instance's host (i.e. hypervisor type and version)
        passes live migration checks (call using amqp rpc into nova manager check_can_live_migrate_destination)
    else destination host not provided, find a candidate destination host and check that it...
        is compatible with the instance's host (i.e. hypervisor type and version)
        passes live migration checks (call using amqp rpc into nova manager check_can_live_migrate_destination)
    call using amqp rpc into nova manager live_migration
    Note: Migration data is initially set by check_can_live_migrate_destination and can be used for implementation specific parameters from this point.
nova.compute.manager.check_can_live_migrate_destination()
    driver.check_can_live_migrate_destination()
    call using amqp rpc into nova manager check_can_live_migrate_source
    driver.check_can_live_migrate_destination_cleanup()
nova.compute.manager.check_can_live_migrate_source()
    determine if the instance is volume backed and add result to the migration data
    driver.check_can_live_migrate_source()
nova.compute.manager.live_migration()
    if block migration request then driver.get_instance_disk_info()
    call using amqp rpc into nova manager pre_live_migration
        Error handler: _rollback_live_migration
    driver.live_migration()
nova.compute.manager.pre_live_migration()
    get the block device information for the instance
    get the network information for the instance
    driver.pre_live_migration()
    setup networks on destination host by calling the network API setup_networks_on_host
    driver.ensure_filtering_rules_for_instance()
nova.compute.manager._rollback_live_migration()
    update instance state to ACTIVE state
    re-setup networks on source host by calling the network API setup_networks_on_host
    for each instance volume connection call using amqp rpc into nova manager remove_volume_connection
    if block migration or volume backed migration without shared storage
        call using amqp rpc into nova manager rollback_live_migration_at_destination
nova.compute.manager._post_live_migration()
    driver.get_volume_connector()
    for each instance volume connection call the volume API terminate_connection
    driver.unfilter_instance()
    call into conductor to network_migrate_instance_start which will eventually call the network API migrate_instance_start
    call using amqp rpc into nova manager post_live_migration_at_destination
    if block migration or not shared storage driver.destroy()
    else driver.unplug_vifs()
    tear down networks on source host by calling the network API setup_networks_on_host
nova.compute.manager.post_live_migration_at_destination()
    setup networks on destination host by calling the network API setup_networks_on_host
    call into conductor to network_migrate_instance_finish which will eventually call the network API migrate_instance_finish
    driver.post_live_migration_at_destination()
    update instance to ACTIVE state
    setup networks on destination host by calling the network API setup_networks_on_host
nova.compute.manager.rollback_live_migration_at_destination()
    tear down networks on destination host by calling the network API setup_networks_on_host
    driver.destroy()
nova.compute.manager.remove_volume_connection()
    call _detach_volume
    driver.get_volume_connector()
    remove the volume connection by calling the volume API terminate_connection
nova.compute.manager._detach_volume()
    driver.detach_volume()
        Since the live migration failed the VM should not be on the destination host.  So this should be a no-op.
    If there is an exception detaching the volume then rollback the detach by calling the volume API roll_detaching
 ```
**Cold Migration Flow**:
```
nova.api.openstack.compute.servers._resize()
nova.api.openstack.compute.contrib.admin_actions._migrate()
nova.compute.api.resize()
    if flavor_id is not passed, migrate host only and keep the original flavor
    else flavor_id is given, migrate host and resize to new flavor
    lookup the image for the instance by calling the image API show
    check quota headroom and reserve
    update instance to RESIZE_PREP state
    determine if the instance's current host should be ignored as a migration target and update filter properties for the scheduler accordingly
    call into scheduler to prep_resize
nova.scheduler.manager.prep_resize()
    call scheduler driver to schedule_prep_resize
    if no valid host was found then update instance to ACTIVE state and rollback quota reservation
    if error occurred then update instance to ERROR state and rollback quota reservation
nova.scheduler.filter_scheduler.schedule_prep_resize()
    run through scheduler filters to select host
    call using amqp rpc into nova manager prep_resize
nova.compute.manager.prep_resize()
    if no node specified call driver.get_available_nodes()
    call _prep_resize
        if an exception occurs then call into scheduler to prep_resize again if possible
nova.compute.manager._prep_resize()
    if same host is used then ensure that the same host is allowed (as per configuration)
    call using amqp rpc into nova manager resize_instance
nova.compute.manager.resize_instance()
    get network and instance information
    update instance to RESIZE_MIGRATING state
    get block device information
    call driver.migrate_disk_and_power_off()
    call _terminate_volume_connections
    call into conductor to network_migrate_instance_start which will eventually call the network API migrate_instance_start
    update instance to RESIZE_MIGRATED state
    call using amqp rpc into nova manager finish_resize
nova.compute.manager._terminate_volume_connections()
    if there is a volume connection to terminate
        driver.get_volume_connector()
        for each volume connection remove the connection by calling the volume API terminate_connection
nova.compute.manager.finish_resize()
    call _finish_resize
    if successful commit the quota reservation
    else rollback the quota reservation and update instance to ERROR state
nova.compute.manager._finish_resize()
    if the flavor is changing then update the instance with the new flavor
    setup networks on destination host by calling the network API setup_networks_on_host
    call into conductor to network_migrate_instance_finish which will eventually call the network API migrate_instance_finish
    update instance to RESIZE_FINISHED state
    refresh and get block device information
    driver.finish_migration()
    update instance to RESIZED state
```

**Cold migration confirm flow**:
```
nova.api.openstack.compute.servers._action_confirm_resize()
nova.compute.api.confirm_resize()
    reserve quota for decrease in resource usage
    call amqp rpc into nova manager confirm_resize
nova.compute.manager.confirm_resize()
    tear down networks on source host by calling the network API setup_networks_on_host
    driver.confirm_migration()
    update instance to ACTIVE (or possibly STOPPED) state
    commit the quota reservation
```
**Cold migration revert flow**:
```
nova.api.openstack.compute.servers._action_revert_resize()
nova.compute.api.revert_resize()
    reserve quota for increase in resource usage
    update instance task state to RESIZE_REVERTING
    call amqp rpc into nova manager revert_resize
nova.compute.manager.revert_resize()
    tear down networks on destination host by calling the network API setup_networks_on_host
    call into conductor to network_migrate_instance_start which will eventually call the network API migrate_instance_start
    get block device information
    driver.destroy()
    call _terminate_volume_connections
    drop resize resources claimed on destination
    call amqp rpc into nova manager finish_revert_resize
nova.compute.manager.finish_revert_resize()
    update instance back to pre-resize values
    re-setup networks on source host by calling the network API setup_networks_on_host
    refresh and get block device information
    driver.finish_revert_migration()
    update instance to RESIZE_REVERTING state
    call into conductor to network_migrate_instance_finish which will eventually call the network API migrate_instance_finish
    update instance to ACTIVE (or possibly STOPPED) state
    commit the quota usage
```
热迁移不关心存储,仅处理OS的相关迁移.冷迁移在分布式存储时,也不关心存储,若为本地存储时,也需要迁移存储相关.<br/>
_Attention:本地存储的instance,不支持热迁移._

Source: http://bodenr.blogspot.com/2014/03/openstack-nova-vm-migration-live-and.html