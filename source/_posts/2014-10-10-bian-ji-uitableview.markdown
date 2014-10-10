---
layout: post
title: "UITableView的编辑－－删除，插入及重新排序"
date: 2014-10-10 16:25:42 +0800
comments: true
categories: iOS开发
---
这两天在开发表情商城的过程中，涉及到了已下载表情包的管理，因此自然想到使用 UITableView 来实现相关的功能，在此，对 UITableView 的编辑－－删除及重新排序进行一下简单的总结和归纳：  
<!-- more -->
##**删除/插入Cell：**  
相关函数：  
{%codeblock%}- (void)setEditing:(BOOL)editing animated:(BOOL)animated;
- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath;
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath;
- (void)insertRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;
- (void)deleteRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;
- (void)beginUpdates;
- (void)endUpdates;{%endcodeblock%}  

具体的流程可以参照下图：  

![table_view_editing](http://imnicblogphotos.qiniudn.com/table_view_editing.jpg)  

下面就这个流程图详细的解释一下：  
###方法一：
1. 用户点击编辑按钮时，此时分两种情况：
	* 我们可以给tableView发送 setEditing:animated: 消息。 
    
    * 当ViewController可以manage一个tableView时，比如，UITableViewController,或者在一个UIViewController中的loadView方法中，直接将一个tableView作为controller的view的情况下，当我们点击Navigation Bar上的Edit Button的时候，controller则会自动发送该消息给tableView。 

2. 当tableView收到 setEditing:animated: 消息之后,tableView会掉用 tableView:canEditRowAtIndexPath: 方法，来进一步的确定哪些rows可以被编辑。补充：

	* 你也可以不实现该方法，在这种情况下，所有的rows都将可以被编辑。  
	
	* 而那些被指定了不能被编辑的rows将会忽略掉UITableViewCell的 editingStyle属性，当然也不会在处于编辑状态时向后缩进。  
	
	* 如果你想让rows可以被编辑，但是又不想显示出编辑按钮如删除或增加的话，你可以实现 tableView:editingStyleForRowAtIndexPath: 方法，然后返回 UITableViewCellEditingStyleNone 。  

3. 接下来tableView会调用 tableView:editingStyleForRowAtIndexPath: 来确定编辑按钮的样式，可以是删除或者增加，当然如果你不实现该回调的话，默认的为UITableViewCellEditingStyleDelete。  
4. 当用户点击左侧的编辑按钮（删除或者增加）的时候，右侧会出现删除按钮，点击右侧的删除按钮则表示确认删除。  
 
5. 最后tableView会发送 tableView:commitEditingStyle:forRowAtIndexPath:给其 datasource，尽管该protocol是可选的，但是如果如果你想删除或者增加一行的话，你就必须实现该方法。在该方法中你必须做以下几件事：  
	+ beginUpdates:当你想要在删除某一行的时候同时以动画的方式展现给用户的话，你必须要先调用该方法，该方法是跟endUpdates配对使用的，在此之间，你不能调用reloadData方法，否则，你必须自己实现动画的展现形式。
	
	+ 发送 deleteRowsAtIndexPaths:withRowAnimation: 或 insertRowsAtIndexPaths:withRowAnimation: 来展示删除。  
	
	+ 更新你的 data-model，比如在array中增加或者删除object。 
	
	+ endUpdates:跟 beginUpdates 配对使用。  
	
	**注意:在endUpdates之前,如果你调用了 deleteRowsAtIndexPaths:withRowAnimation:时，
	一定记得要更新数据，比如删除新的数据在model－data中，否则应用将会崩溃，报错：  
	Assertion failure in -[UITableView _endCellAnimationsWithContext:]**。

###方法二：
除此之外，你也可以使用swipe-to-delete方法，但是具体的流程跟上面不太相同：  

1. 当用户在一个cell上滑动的时候tableView会检测data source是否实现了 tableView:commitEditingStyle:forRowAtIndexPath: 方法，如果实现了的话他就会发送 setEditing:animated: 模式，然后进入编辑模式。  
**注意：在此方法中,你不能调用setEditing:animated:方法，如果由于某种原因必须要调用的话，你必须使用performSelector:withObject:afterDelay:方法来调用**。  

2. 当用户开始滑动的时候会调用tableView:willBeginEditingRowAtIndexPath: ，当你点击删除或者点击空白处取消的时候则会调用 tableView:didEndEditingRowAtIndexPath:方法。  

3. 接下来你只需要在 tableView:commitEditingStyle:forRowAtIndexPath: 方法中实现跟上面一样的方法即可。

###方法三：  
当然，你也可以不采用系统原生的编辑按钮，你可以自己在cell中自定义一个删除按钮，当用户点击删除按钮的时候，你只需要做如下的事情即可：  

{%codeblock%} [self.tableView beginUpdates];  
[self.tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];  
[self.resourceArray removeObjectAtIndex:indexPath.row];  
[self.tableView endUpdates];{%endcodeblock%}  
    
当然你也可以直接：  

{%codeblock%}[self.resourceArray removeObjectAtIndex:indexPath.row];    
[self.tableView reloadData];{%endcodeblock%}  

但是这样用户体验并不好，不推荐！  

而对于插入Cell的话，虽然我们也可以使用类似删除的系统原生方式（当tableView处于编辑模式时，点击cell左侧的编辑/插入按钮），但我们更倾向于使用在navigation bar上添加一个按钮，而具体的过程可以参照上面删除的流程。  

##**重新排序**  
清楚了上面的流程之后我们在做排序的时候，我们在去做重新排序的时候就会比较简单了。  
相关函数：  
{%codeblock%}- (void)setShowsReorderControl:(BOOL)showsReorderControl;  
- (void)setEditing:(BOOL)editing animated:(BOOL)animate;  
- (BOOL)tableView:(UITableView *)tableView canMoveRowAtIndexPath:(NSIndexPath *)indexPath;  
- (NSIndexPath *)tableView:(UITableView *)tableView targetIndexPathForMoveFromRowAtIndexPath:(NSIndexPath *)sourceIndexPath toProposedIndexPath:(NSIndexPath *)proposedDestinationIndexPath  
- (void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)fromIndexPath toIndexPath:(NSIndexPath *)toIndexPath{%endcodeblock%}  

具体的流程可以参照下图：  

![table_view_moving](http://imnicblogphotos.qiniudn.com/table_view_moving.jpg)  

同样的，我们对这个流程也进行一下详细的解释：  

1. 首先，当用户点击排序按钮时，tableView进入编辑模式，这也是分两种情况： 
 
	* 我们可以给tableView发送 setEditing:animated: 消息。 
    
    * 当ViewController可以manage一个tableView时，比如，UITableViewController,或者在一个UIViewController中的loadView方法中，直接将一个tableView作为controller的view的情况下，当我们点击Navigation Bar上的Edit Button的时候，controller则会自动发送该消息给tableView。 

2. 当tableView收到 setEditing:animated: 消息之后，他会给data source发送tableView:canMoveRowAtIndexPath:方法，在该方法中你可以筛选出来那些cell是可以被move而哪些不可以。  

3. 当用户拖动某一个cell上下移动的时候，就会调用  

    tableView:targetIndexPathForMoveFromRowAtIndexPath:toProposedIndexPath:  

    这个方法中，我们可以指定cell的可移动范围，比如：  
{%codeblock%}- (NSIndexPath *)tableView:(UITableView *)tableView
        targetIndexPathForMoveFromRowAtIndexPath:(NSIndexPath *)sourceIndexPath
        toProposedIndexPath:(NSIndexPath *)proposedDestinationIndexPath   
{
    NSDictionary *section = [data objectAtIndex:sourceIndexPath.section];  
    NSUInteger sectionCount = [[section valueForKey:@"content"] count];  
    if (sourceIndexPath.section != proposedDestinationIndexPath.section)   
    {  
        NSUInteger rowInSourceSection =  
             (sourceIndexPath.section > proposedDestinationIndexPath.section) ?  
               0 : sectionCount - 1;  
        return [NSIndexPath indexPathForRow:rowInSourceSection   inSection:sourceIndexPath.section];  
    }   
    else if (proposedDestinationIndexPath.row >= sectionCount)   
    {  
        return [NSIndexPath indexPathForRow:sectionCount - 1   inSection:sourceIndexPath.section];  
    }  
    // Allow the proposed destination.  
    return proposedDestinationIndexPath;  
}{%endcodeblock%}  

4. 最后，当用户松开手之后，tableView就会给data source发送 tableView:moveRowAtIndexPath:toIndexPath: 消息，在该方法中我们可以更新model－data，比如：  
{%codeblock%}- (void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath *)destinationIndexPath   
{  
    NSString *stringToMove = [self.reorderingRows objectAtIndex:sourceIndexPath.row];  
    [self.reorderingRows removeObjectAtIndex:sourceIndexPath.row];  
    [self.reorderingRows insertObject:stringToMove atIndex:destinationIndexPath.row];  
}{%endcodeblock%}  

**注意：我们如果想要显示reorder control，就必须设置cell 的 showsReorderControl属性为YES,同时必须实现tableView:moveRowAtIndexPath:toIndexPath: 方法**。  
