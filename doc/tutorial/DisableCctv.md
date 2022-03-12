# Disable all CCTV

Quick helper method to disable all security cameras. 

```
void disable_cctv()
{
    struct PropRecord *prop;
    
    prop = get_ptr_obj_pos_list_current_entry();

    for (; prop != NULL; prop = prop->prev)
    {
        if (prop->type == PROP_TYPE_OBJ && prop->obj != NULL)
        {
            struct ObjectRecord *obj = prop->obj;
            if (obj->type == PROPDEF_CCTV)
            {
                struct CCTVRecord *cctv = (struct CCTVRecord *)obj;
                cctv->flags |= PROPFLAG_WEAPON_LEFTHANDED;
            }
        }
    }
}
```