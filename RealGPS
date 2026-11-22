#import <Foundation/Foundation.h>
#import <CoreLocation/CoreLocation.h>
#import <UIKit/UIKit.h>
#import <objc/runtime.h>

static NSString *const kFakeGPSEnabledKey = @"FakeGPS_Enabled";
static NSString *const kFakeGPSLatKey = @"FakeGPS_Lat";
static NSString *const kFakeGPSLonKey = @"FakeGPS_Lon";

@interface FakeGPSManager : NSObject
@property (nonatomic, strong) UIButton *floatingButton;
+ (instancetype)shared;
- (void)setupFloatingButton;
@end

@implementation FakeGPSManager

+ (instancetype)shared {
    static FakeGPSManager *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc] init];
    });
    return instance;
}

- (UIWindow *)activeKeyWindow {
    for (UIScene *scene in [UIApplication sharedApplication].connectedScenes) {
        if (scene.activationState == UISceneActivationStateForegroundActive &&
            [scene isKindOfClass:[UIWindowScene class]]) {
            UIWindowScene *windowScene = (UIWindowScene *)scene;
            for (UIWindow *window in windowScene.windows) {
                if (window.isKeyWindow) {
                    return window;
                }
            }
            return windowScene.windows.firstObject;
        }
    }
    return nil;
}

- (void)setupFloatingButton {
    if (self.floatingButton && self.floatingButton.superview) return;
    UIWindow *window = [self activeKeyWindow];
    if (!window) return;

    UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
    btn.frame = CGRectMake(20, 120, 44, 44);
    btn.layer.cornerRadius = 22;
    btn.backgroundColor = [[UIColor blackColor] colorWithAlphaComponent:0.65];
    btn.layer.borderWidth = 1.5;
    btn.layer.borderColor = [UIColor whiteColor].CGColor;
    [btn setTitle:@"📍" forState:UIControlStateNormal];
    btn.titleLabel.font = [UIFont systemFontOfSize:20];

    [btn addTarget:self action:@selector(showLocationPrompt) forControlEvents:UIControlEventTouchUpInside];

    UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handlePan:)];
    [btn addGestureRecognizer:pan];

    [window addSubview:btn];
    self.floatingButton = btn;
    [self updateButtonAppearance];
}

- (void)updateButtonAppearance {
    BOOL enabled = [[NSUserDefaults standardUserDefaults] boolForKey:kFakeGPSEnabledKey];
    self.floatingButton.layer.borderColor = enabled ? [UIColor systemGreenColor].CGColor : [UIColor whiteColor].CGColor;
}

- (void)handlePan:(UIPanGestureRecognizer *)pan {
    UIWindow *window = [self activeKeyWindow];
    if (!window) return;
    CGPoint translation = [pan translationInView:window];
    pan.view.center = CGPointMake(pan.view.center.x + translation.x, pan.view.center.y + translation.y);
    [pan setTranslation:CGPointZero inView:window];
}

- (void)showLocationPrompt {
    UIWindow *window = [self activeKeyWindow];
    UIViewController *topVC = window.rootViewController;
    while (topVC.presentedViewController) {
        topVC = topVC.presentedViewController;
    }
    if (!topVC) return;

    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    BOOL isEnabled = [defaults boolForKey:kFakeGPSEnabledKey];
    double savedLat = [defaults objectForKey:kFakeGPSLatKey] ? [defaults doubleForKey:kFakeGPSLatKey] : 10.7769;
    double savedLon = [defaults objectForKey:kFakeGPSLonKey] ? [defaults doubleForKey:kFakeGPSLonKey] : 106.7009;

    NSString *status = isEnabled ? @"Đang BẬT" : @"Đang TẮT";
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Fake GPS"
                                                                   message:[NSString stringWithFormat:@"Trạng thái: %@\nNhập tọa độ (ví dụ lấy từ Google Maps):", status]
                                                            preferredStyle:UIAlertControllerStyleAlert];

    [alert addTextFieldWithConfigurationHandler:^(UITextField *textField) {
        textField.placeholder = @"Vĩ độ (Latitude) - VD: 10.7769";
        textField.keyboardType = UIKeyboardTypeDecimalPad;
        textField.text = [NSString stringWithFormat:@"%.6f", savedLat];
    }];

    [alert addTextFieldWithConfigurationHandler:^(UITextField *textField) {
        textField.placeholder = @"Kinh độ (Longitude) - VD: 106.7009";
        textField.keyboardType = UIKeyboardTypeDecimalPad;
        textField.text = [NSString stringWithFormat:@"%.6f", savedLon];
    }];

    UIAlertAction *applyAction = [UIAlertAction actionWithTitle:@"Bật Fake GPS"
                                                          style:UIAlertActionStyleDefault
                                                        handler:^(UIAlertAction *action) {
        NSString *latStr = [alert.textFields[0].text stringByReplacingOccurrencesOfString:@"," withString:@"."];
        NSString *lonStr = [alert.textFields[1].text stringByReplacingOccurrencesOfString:@"," withString:@"."];
        [defaults setDouble:[latStr doubleValue] forKey:kFakeGPSLatKey];
        [defaults setDouble:[lonStr doubleValue] forKey:kFakeGPSLonKey];
        [defaults setBool:YES forKey:kFakeGPSEnabledKey];
        [defaults synchronize];
        [self updateButtonAppearance];
    }];

    UIAlertAction *disableAction = [UIAlertAction actionWithTitle:@"Tắt (Dùng GPS thật)"
                                                            style:UIAlertActionStyleDestructive
                                                          handler:^(UIAlertAction *action) {
        [defaults setBool:NO forKey:kFakeGPSEnabledKey];
        [defaults synchronize];
        [self updateButtonAppearance];
    }];

    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"Hủy" style:UIAlertActionStyleCancel handler:nil];

    [alert addAction:applyAction];
    [alert addAction:disableAction];
    [alert addAction:cancelAction];

    [topVC presentViewController:alert animated:YES completion:nil];
}

@end

@implementation CLLocation (FakeGPS)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method orig = class_getInstanceMethod(self, @selector(coordinate));
        Method swizzled = class_getInstanceMethod(self, @selector(fake_coordinate));
        if (orig && swizzled) {
            method_exchangeImplementations(orig, swizzled);
        }

        [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationDidBecomeActiveNotification
                                                          object:nil
                                                           queue:[NSOperationQueue mainQueue]
                                                      usingBlock:^(NSNotification *note) {
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                [[FakeGPSManager shared] setupFloatingButton];
            });
        }];
    });
}

- (CLLocationCoordinate2D)fake_coordinate {
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    if ([defaults boolForKey:kFakeGPSEnabledKey]) {
        double lat = [defaults doubleForKey:kFakeGPSLatKey];
        double lon = [defaults doubleForKey:kFakeGPSLonKey];
        return CLLocationCoordinate2DMake(lat, lon);
    }
    return [self fake_coordinate];
}

@end
