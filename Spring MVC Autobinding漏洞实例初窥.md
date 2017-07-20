### ����
���������������ʱ��nowillҲ����֪��������һƪ�����Զ���©�������¡�[ǳ���Զ���©��](https://xianzhi.aliyun.com/forum/read/1801.html)��������ϸ������Ȥ�Ŀ��Կ��¡����Ա�ƪ�����ص��ʵ��������Spring MVC Autobinding©����
 
Autobinding-�Զ���©�������ݲ�ͬ����/��ܣ���©���м�����ͬ�Ľз������£�
 
* Mass Assignment: Ruby on Rails, NodeJS
* Autobinding: Spring MVC, ASP.NET MVC
* Object injection: PHP(����ע�롢�����л�©��)
 
��������ʱ��������Ա�Զ���HTTP��������󶨵�����������������У��Ӷ�ʹ������Ա�����׵�ʹ�øÿ�ܡ����﹥���߾Ϳ����������ַ���ͨ������http���󣬽���������󶨵������ϣ��������߼�ʹ�øö������ʱ�Ϳ��ܲ���һЩ����Ԥ�ϵĽ������Ϊ���ｲSpring MVC��ܣ�������������Ϊ�Զ���©������Щ��Ϣ�����Բο�owasp�Ϲ���[Mass Assignment](https://www.owasp.org/index.php/Mass_Assignment_Cheat_Sheet#Spring_MVC)�Ľ��ܡ�
 
���´���ʵ����[ZeroNights-HackQuest-2016](https://github.com/GrrrDog/ZeroNights-HackQuest-2016)��demoΪ��
 
### @ModelAttributeע��
 
��Spring mvc�У�ע��@ModelAttribute��һ���ǳ����õ�ע�⣬�书����Ҫ�������棺
 
* �����ڲ����ϣ��Ὣ�ͻ��˴��ݹ����Ĳ���������ע�뵽ָ�������У����һὫ��������Զ�����ModelMap�У�����View��ʹ�ã�
* �����ڷ����ϣ�����ÿһ��@RequestMapping��ע�ķ���ǰִ�У�����з���ֵ�����Զ����÷���ֵ���뵽ModelMap�У�
 
@ModelAttributeע��һ�������Ĳ���,��Form����URL�����л�ȡ
 
``` java
 
@RequestMapping(value = "/home", method = RequestMethod.GET)
    public String home(@ModelAttribute User user, Model model) {
        if (showSecret){
            model.addAttribute("firstSecret", firstSecret);
        }
        return "home";
    }
 
```
 
view��ͨ��${user.name}���ɷ��ʡ�ע����ʱ�����User��һ��Ҫ��û�в����Ĺ��캯�������磺
 
``` java
public class User {
 
    private String name;
    private String pass;
    private Integer weight;
 
    public User() {
 
    }
 
    public User(String name, String pass, Integer weight) {
        this.name=name;
        this.pass=pass;
        this.weight=weight;
    }
    ......
}
```
 
@ModelAttributeע��һ������,�÷������ڴ�controllerÿ��@RequestMapping����ִ��ǰ��ִ��
 
```
    @ModelAttribute("showSecret")
    public Boolean getShowSectet() {
        logger.debug("flag: " + showSecret);
        return showSecret;
    }
```
### @SessionAttributesע��
 
��Ĭ������£�ModelMap �е������������� request ����Ҳ����˵�����������������ModelMap �е����Խ����١����ϣ���ڶ�������й��� ModelMap �е����ԣ����뽫������ת�浽 session �У����� ModelMap �����Բſ��Ա���������ʡ�
 
Spring ����������ѡ���ָ�� ModelMap �е���Щ������Ҫת�浽 session �У��Ա���һ�������Ӧ�� ModelMap �������б��л��ܷ��ʵ���Щ���ԡ���һ������ͨ���ඨ�崦��ע @SessionAttributes("user") ע����ʵ�ֵġ�SpringMVC �ͻ��Զ��� @SessionAttributes ���������ע�뵽 ModelMap ������ setup action �Ĳ����б�ʱ��ȥ ModelMap ��ȡ�������Ķ�������ӵ������б�ֻҪ��ȥ���� SessionStatus �� setComplete() �������������ͻ�һֱ������ Session �У��Ӷ�ʵ�� Session ��Ϣ�Ĺ���
 
### justiceleague ʵ�����
�ѳ����������������Կ������Ӧ�ò˵�����about��reg��Sign up��    Forgot password����4��ҳ����ɡ����ǹ�ע���ص��������һع��ܣ�����ô���ƹ���ȫ������֤���һ����롣�������ǹ�ע���ص�����ô���ƹ������һع��ܡ�
 
1�����ȿ�reset�������Ѳ�Ӱ������߼���ɾ��������������׶���
 
```
@Controller
@SessionAttributes("user")
public class ResetPasswordController {
 
private UserService userService;
...
@RequestMapping(value = "/reset", method = RequestMethod.POST)
public String resetHandler(@RequestParam String username, Model model) {
        User user = userService.findByName(username);
        if (user == null) {
            return "reset";
        }
        model.addAttribute("user", user);
        return "redirect: resetQuestion";
    }
```
����Ӳ�����ȡusername�������û������û��������������user����ŵ�Model�С���Ϊ���Controllerʹ����@SessionAttributes("user")������ͬʱҲ���Զ���user����ŵ�session�С�Ȼ����ת��resetQuestion�����һذ�ȫ����У��ҳ�档
 
Ϊʲô������Զ���user����ŵ�session�У�����ԭ���@SessionAttributesע��
 
 
2��resetQuestion�����һذ�ȫ����У��ҳ����resetViewQuestionHandler�������չ��
 
```
@RequestMapping(value = "/resetQuestion", method = RequestMethod.GET)
    public String resetViewQuestionHandler(@ModelAttribute User user) {
        logger.info("Welcome resetQuestion ! " + user);
        return "resetQuestion";
    }
```
����ʹ����@ModelAttribute User user��ʵ���������Ǵ�session�л�ȡuser���󡣵�������������������������user����ĳ�Ա����ʱ������user�����Ӧ��Ա��ֵ��
���Ե����Ǹ�resetQuestionHandler����GET�����ʱ�������ӡ�answer=hehe�������������Ϳ��Ը�session�еĶ���ֵ����ԭ�������һصİ�ȫ������޸ĳɡ�hehe�������������һ��У�鰲ȫ����ʱ������֤�ɹ����һ�����
 
### ��ȫ����
 
Spring MVC�п���ʹ��@InitBinderע�⣬ͨ��WebDataBinder�ķ���setAllowedFields��setDisallowedFields�������������󶨵Ĳ�����
 
### �ο�
 
[1] https://www.owasp.org/index.php/Mass_Assignment_Cheat_Sheet#Spring_MVC
[2] http://bobao.360.cn/learning/detail/3991.html
[3] https://github.com/GrrrDog/ZeroNights-HackQuest-2016