# FXOSNodes

**Note: Reference Source Only**

*This source is provided to demostrate the node list system.  It will not compile, as is, but the node/linked-list functionality can be inspected.  One could remove the OS specific functionality and use the "nodelist" functionality.*


***Below ae the main nodelist methods.  The names are generally self explanatory.  Code comments will be added to each at a leater date, as the functionlity is made permanent.*** 

```
// PFXNODE - Basic Linked-List 
// PFXNODELIST - Wrapped Link-List with additional features
PFXNODE k_nodelist_init(BYTE type,LPCSTR name,LPVOID data);
PFXNODE k_nodelist_create(BYTE type,LPCSTR name,LPVOID data,PFXNODE next,PFXNODE last);
PFXNODE k_nodelist_ncreate(BYTE type,ULONG objId,LPVOID data,PFXNODE last,PFXNODE next);
PFXNODE k_nodelist_copy(PFXNODE node);
VOID    k_nodelist_add(PFXNODE head,PFXNODE new);
VOID 	k_nodelist_addtohead(PFXNODE head,PFXNODE new);
PFXNODE k_nodelist_gettype(PFXNODE head,BYTE type);
PFXNODE k_nodelist_get(PFXNODE head,INT index);
PFXNODE k_nodelist_getname(PFXNODE head,LPCSTR name);
PFXNODE k_nodelist_getname_and_type(PFXNODE head,LPCSTR name,BYTE type);
PFXNODE k_nodelist_remove(PFXNODE head,LPCSTR name);
PFXNODE k_nodelist_remove_node(PFXNODE head,PFXNODE targetNode);
PFXNODE k_nodelist_remove_obj(PFXNODE head,ULONG objId);
PFXNODE	k_nodelist_last(PFXNODE head);

VOID k_deallocate_default(LPCSTR name,LPVOID data);

PFXNODELIST k_nodelist_allocate_list(LPCSTR listName,NodeListDeallocator deallocator);
VOID		k_nodelist_deallocate_list(PFXNODELIST nodelist);
PFXNODE		k_nodelist_addtolist(PFXNODELIST nodelist,BYTE type,LPCSTR name,LPVOID data);
PFXNODE 	k_nodelist_naddtolist_tohead(PFXNODELIST list,BYTE type,ULONG objId,LPVOID data);
PFXNODE 	k_nodelist_addtolist_tohead(PFXNODELIST list,BYTE type,LPCSTR name,LPVOID data);
PFXNODE 	k_nodelist_naddtolist(PFXNODELIST list,BYTE type,ULONG objId,LPVOID data);
PFXNODE 	k_nodelist_addnodetolist(PFXNODELIST list,PFXNODE new);
PFXNODELIST	k_nodelist_clear_list(PFXNODELIST nodelist);
PFXNODE		k_nodelist_searchByName(PFXNODELIST list,LPCSTR name);
PFXNODE		k_nodelist_searchByType(PFXNODELIST list,BYTE type);
LPCSTR		k_nodelist_getlistname(PFXNODELIST list);
PFXNODE		k_nodelist_getfirstnode(PFXNODELIST list);
BOOL		k_nodelist_empty(PFXNODELIST list);

typedef void (*FOREACHNODE)(LPVOID ctx,LPVOID pdata);
typedef BOOL (*FOREACHNODEUNTIL)(LPVOID ctx,LPVOID pdata);


VOID    k_nodelist_foreach_data(PFXNODE head,LPVOID ctx,FOREACHNODE each);
PFXNODE k_nodelist_foreach_until_data(PFXNODE head,LPVOID context,FOREACHNODEUNTIL each);
VOID    k_nodelist_foreach_type(PFXNODELIST list,BYTE type,LPVOID context,FOREACHNODE each); // TODO: fix name
VOID    k_nodelist_foreach_listdata(PFXNODELIST list,LPVOID context,FOREACHNODE each);
PFXNODE k_nodelist_foreach_until_listdata(PFXNODELIST list,LPVOID context,FOREACHNODEUNTIL each);
VOID 	k_nodelist_foreach_listdata_remove(PFXNODELIST list,LPVOID context,FOREACHNODEUNTIL checkStatus);

```

*Examples of Use*

## Allocate a list
```
_k_windowManager_WindowClassList 	= k_nodelist_allocate_list("_window_class_list" ,k_deallocate_window_class);
```

*Deallocation Function*
```
VOID k_deallocate_window_class(LPCSTR name,LPVOID data)
{
	PWINDOW pWin = NULL;
	k_debug_strings("k_deallocate_window_class name:",(LPSTR)name);
	k_debug_pointer("                            ptr:",data);

	k_mem_deallocate_heap(data);
}
```

## Deallocate a list
```
VOID k_deallocate_window_object(LPCSTR name,LPVOID data)
{
	PWINDOW pWin = NULL;
	k_debug_strings("k_deallocate_window_object name:",(LPSTR)name);
	k_debug_pointer("                            ptr:",data);

	if(data)
	{
		pWin = (PWINDOW)data;
		k_debug_strings("                        caption:",(LPSTR)pWin->win_title);


		if(pWin->pChildHitList)
		{
			k_nodelist_deallocate_list(pWin->pChildHitList);
		}
		if(pWin->pChildWindows)
		{
			k_nodelist_deallocate_list(pWin->pChildWindows);
		}
		if(pWin->windowData)
			k_mem_deallocate_heap(pWin->windowData);


		k_mem_deallocate_heap(pWin);
	}

}
```

## FOREACH Functionality
*Iterate a nodelist and apply a data check callback for a condition (TRUE/FALSE)*
```
HWND k_user_findWindow(LPCSTR winTitle)
{
	PFXNODE node = NULL;
	HWND hWnd = NULL;

	node = k_nodelist_foreach_until_listdata(_k_windowManager_WindowObjectList,(LPVOID)winTitle,find_window_caption);
	if(node)
	{
		hWnd = k_getHandleFromWindow((PWINDOW)node->data);
	}
	return hWnd;

}
```

*FOREACH Callback*
```
BOOL find_window_caption(LPVOID ctx,LPVOID data)
{
	BOOL bRet = FALSE;

	if(ctx && data)
	{
		if(strcmp((LPCHAR)ctx,((PWINDOW)data)->win_title) == 0)
			bRet = TRUE;
	}

	return bRet;
}
```
