# SnackbarHelper
这个类用于管理Snackbar。隐藏Android样板代码，并公开更简单的方法

# 公共方法
### public boolean isShowing()

返回snackbar是否显示

### public void showMessage(Activity activity, String message)

显示特定消息的Snackbar

### public void showMessageWithDismiss(Activity activity, String message)

显示特定消息的和dismiss按钮的Snackbar

### public void showError(Activity activity, String errorMessage)

显示给定错误消息的Snackbar。 按下dismiss时，将结束activity
对于通知错误很有用，在这种情况下无法与活动进行进一步的交互

### public void hide(Activity activity)

如果当前有显示的Snackbar，隐藏它。可以从任何线程安全地调用，即使Snackbar没有显示