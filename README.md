Yii2-Alipay

֧������ʱ���˵�Yii2��չ������

Install With Composer

The preferred way to install this extension is through composer.

Either run

php composer.phar require lhq5189/alipay "*"
or for the dev-master

php composer.phar require lhq5189/alipay "dev"
Or, you may add

"lhq5189/alipay": "*"
to the require section of your composer.json file and execute php composer.phar update.

Usage

Once the extension is installed, simply modify your application configuration as follows:

return [
    'alipayPartner' => '2088000000000000',
    'alipaySellerEmail' => 'xxx@xxx.com.cn',
    'alipayKey' => 'j8zjlejjebpgei98cbbgbbmwfr4asdf',
];
AlipayController

���Ǽ�ʱ�������ӣ�֧���������ӿڻ�����Ӧ�Ľӿڼ���ʹ��

namespace frontend\controllers;

use app\models\Test_pay;
use yii\log\Logger;
use app\models\Test2;
use lhq5189\alipay\AlipayNotify;
use yii\web\Controller;
use lhq5189\alipay\AlipayConfig;
use lhq5189\alipay\AlipaySubmit;
use Yii;

class Pay2Controller extends Controller
{
    public $layout='main';
    public function beforeAction($action)
    {
        if ('notify' == $action->id || 'index' == $action->id) {
            $this->enableCsrfValidation = false;
        }
        return parent::beforeAction($action);
    }
    public function actionIndex()
    {
        $data = Yii::$app->request->post();
        $body = $data['name'];
        $total_fee = $data['price'];
        $out_trade_no = $data['id'];
        /**************************�������**************************/

        //�������첽��ͬ��֪ͨҳ��·��
       $notify_url = Yii::$app->urlManager->createAbsoluteUrl(['pay2/notify']);
        $return_url = Yii::$app->urlManager->createAbsoluteUrl(['pay2/return']);
        //��http://��ʽ������·�����������?id=123�����Զ��������yii2Ĭ�ϵ�·�ɷ����޷����յ��첽֪ͨ��Ҫ ��Ϊwww.com/xxx/xxx/��ʽ���ʲſ���


        $alipayConfig = (new AlipayConfig())->getAlipayConfig();

        //����Ҫ����Ĳ������飬����Ķ�
        $parameter = array(
            "service" => "create_direct_pay_by_user",
            "partner" => trim($alipayConfig['partner']),
            "notify_url"    => $notify_url,
            "return_url" =>$return_url,
            'body'=>$body,
            'out_trade_no'=>$out_trade_no,
            'subject'=>$body,
            'payment_type'=>1,
            'show_url'=>'http://www.cfzxzz.com',
            'total_fee'=>$total_fee,
            'seller_id'=>trim($alipayConfig['partner']),
            "_input_charset"    => trim(strtolower($alipayConfig['input_charset']))
        );

        //��������
        $alipaySubmit = new AlipaySubmit($alipayConfig);
        $html_text = $alipaySubmit->buildRequestForm($parameter,"get", "������ת��֧��ҳ��");
        return $html_text;
    }

    public function actionNotify()
    {
        $alipayConfig = (new AlipayConfig())->getAlipayConfig();

        Yii::getLogger()->log("alipay Notify Start", Logger::LEVEL_ERROR);

        Yii::getLogger()->log("��������������������alipayConfig������������������������������", Logger::LEVEL_ERROR);
        Yii::getLogger()->log(print_r($alipayConfig, true), Logger::LEVEL_ERROR);

        $notify = new AlipayNotify($alipayConfig);
        if ($notify->verifyNotify()) {
            $trade_no = Yii::$app->request->post('trade_no');
            $orderid = Yii::$app->request->post('out_trade_no');
            $subject = Yii::$app->request->post('subject');
            $trade_status = Yii::$app->request->post('trade_status');
            $total_fee = Yii::$app->request->post('total_fee');
            $buyer_email = Yii::$app->request->post('buyer_email');
            /**********************************************************************
             * ����Ϊ֧���ɹ����Լ��Ĳ��������Ը��ݾ������д

            $model = new Test2();
            $model->title = $orderid;
            $model->name = $trade_no.'-'.$orderid.'-'.$subject.'-'.$trade_status.'-'.$total_fee.'-'.time().'-'.$buyer_email;
            $model->time = time();
            $model->save();
            $modelcf = Test_pay::findOne($orderid);
            $modelcf->is_pay =1;
            $modelcf->username =$buyer_email;
            $modelcf->save();
             **********************************************************************************/
//            Yii::getLogger()->log('verify Notify success', Logger::LEVEL_ERROR);
            return "success";
        } else {
            Yii::getLogger()->log('verify Notify failed', Logger::LEVEL_ERROR);
            return "fail";
        }
    }

    public function actionReturn(){
        $alipayConfig = (new AlipayConfig())->getAlipayConfig();
        $notify = new AlipayNotify($alipayConfig);
        $notify->verifyNotify();
        $data = Yii::$app->request->get();
        if($data){
            $this->redirect('/testpay/list');   //֧���ɹ�����ת��ַ

        }else{
            echo '֧��ʧ��';
        }
    }


}