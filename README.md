# 프로젝트 설명
+ 오디오 파일을 재생할 수 있다면 벨소리나 알람과 같이 각종 소리와 관련된 다양한 작업을 할 수 있습니다. 오디오를 재생하는 방법 중 가장 쉬운 방법은 AVAudioPlayer를 사용하는 것으로, AVAudioPlayer를 이용하여 오디오 파일을 재생, 일시정지 및 정지하는 방법과 볼륨을 조절하는 방법 그리고 녹음하는 방법을 알아봅시다.

---

# 전체 소스 & 해석
+ ViewController.swift

```swift
import UIKit

import AVFoundation

 

class ViewController: UIViewController, AVAudioPlayerDelegate, AVAudioRecorderDelegate{

    
 
// AVAudioPlayer 인스턴스, 재생할 오디오, 타이머 변수 및 최대 볼륨 상수 선언 
    var audioPlayer : AVAudioPlayer!

    var audioFile : URL!

    

    let MAX_VOLUME : Float = 10.0

    

    var progressTimer : Timer!

    let timePlayerSelector:Selector = #selector(ViewController.updatePlayTime)

    let timeRecordSelector:Selector = #selector(ViewController.updateRecordTime)

 

    @IBOutlet var pvProgressPlay: UIProgressView!

    @IBOutlet var lblCurrentTime: UILabel!

    @IBOutlet var lblEndTime: UILabel!

    @IBOutlet var btnPlay: UIButton!

    @IBOutlet var btnPause: UIButton!

    @IBOutlet var btnStop: UIButton!

    @IBOutlet var slVolume: UISlider!

    

    

    @IBOutlet var btnRecord: UIButton!

    @IBOutlet var lblRecordTime: UILabel!

    

    var audioRecorder : AVAudioRecorder!

    var isRecordMode = false

    

    override func viewDidLoad() {

        super.viewDidLoad()

        // Do any additional setup after loading the view.

        selectAudioFile()

        if !isRecordMode { // 재생 모드일 경우

        initplay()

            btnRecord.isEnabled = false

            lblCurrentTime.isEnabled = false

        } else { // 녹음 모드일 경우

            initRecord()

        }

    }

    
// 모드에 따라 다른 파일 선택
    func selectAudioFile() {

        if !isRecordMode {

            audioFile = Bundle.main.url(forResource: "Sicilian_Breeze", withExtension: "mp3")

        } else {

            let documentDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]

            audioFile = documentDirectory.appendingPathComponent("recordFile.m4a")

            

        }

    }

    
// 녹음 모드 초기화
    func initRecord() {

        let recordSettings = [

            AVFormatIDKey : NSNumber(value: kAudioFormatAppleLossless as UInt32),

            AVEncoderAudioQualityKey : AVAudioQuality.max.rawValue,

            AVEncoderBitRateKey: 320000,

            AVNumberOfChannelsKey : 2,

            AVSampleRateKey : 44100.0] as [String : Any]

        do {

            audioRecorder = try AVAudioRecorder(url: audioFile, settings: recordSettings)

        } catch let error as NSError {

            print("Error-initRecord : \(error)")

        }

        

        audioRecorder.delegate = self

        

        slVolume.value = 1.0

        audioPlayer.volume = slVolume.value

        lblEndTime.text = convertNSTimeInterval2String(0)

        lblCurrentTime.text = convertNSTimeInterval2String(0)

        setPlayButtons(false, pause: false, stop: false)

        

        let session = AVAudioSession.sharedInstance()

        do {

            try AVAudioSession.sharedInstance().setCategory(.playAndRecord, mode: .default)

            try AVAudioSession.sharedInstance().setActive(true)

        } catch let error as NSError {

            print(" Error-setCategory : \(error)")

        }

        do {

            try session.setActive(true)

        } catch let error as NSError {

            print(" Error-setActive : \(error)")

        }

    }

    
// 재생 모드 초기화
    func initplay() {

        do {

            audioPlayer = try AVAudioPlayer(contentsOf: audioFile)

        } catch let error as NSError {

            print("Error-initplay : \(error)")

        }

        slVolume.maximumValue = MAX_VOLUME

        slVolume.value = 1.0

        pvProgressPlay.progress = 0

        

        audioPlayer.delegate = self

        audioPlayer.prepareToPlay()

        audioPlayer.volume = slVolume.value

        

        lblEndTime.text = convertNSTimeInterval2String(audioPlayer.duration)

        lblCurrentTime.text = convertNSTimeInterval2String(0)

        setPlayButtons(true, pause: false, stop: false)

        

    }

    
// 버튼 활성화 or 비활성화 함수
    func setPlayButtons(_ play:Bool, pause:Bool, stop:Bool) {

        btnPlay.isEnabled = play

        btnPause.isEnabled = pause

        btnStop.isEnabled = stop

    }

    
// 00:00 형태의 문자열로 변환
    func convertNSTimeInterval2String(_ time:TimeInterval) -> String{

        let min = Int(time/60)

        let sec = Int(time.truncatingRemainder(dividingBy: 60))

        let strTime = String(format: "%02d:%02d", min, sec)

        return strTime

        

    }

 
// 재생 버튼 선택 시 
    @IBAction func btnPlayAudio(_ sender: UIButton) {

        audioPlayer.play()

        setPlayButtons(false, pause: true, stop: true)

        progressTimer = Timer.scheduledTimer(timeInterval: 0.1, target: self, selector: timePlayerSelector, userInfo: nil, repeats: true)

    }

    
// 0.1초마다 호출 및 재생시간 표시 
    @objc func updatePlayTime() {

        lblCurrentTime.text = convertNSTimeInterval2String(audioPlayer.currentTime)

        pvProgressPlay.progress = Float(audioPlayer.currentTime/audioPlayer.duration)

    }

    
// 버튼 클릭 시 
    @IBAction func btnPauseAudio(_ sender: UIButton) {

        audioPlayer.pause()

        setPlayButtons(true, pause: false, stop: true)

    }

    
// 버튼 클릭 시 
    @IBAction func btnStopAudio(_ sender: UIButton) {

        audioPlayer.stop()

        audioPlayer.currentTime = 0

        lblCurrentTime.text = convertNSTimeInterval2String(0)

        setPlayButtons(true, pause: false, stop: false)

        progressTimer.invalidate()

    }

    
// slVolume.value 대입
    @IBAction func slChangeVolume(_ sender: UISlider) {

        audioPlayer.volume = slVolume.value

    }

    
// 재생 종료 시 호출 
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {

        progressTimer.invalidate()

        setPlayButtons(true, pause: false, stop: false)

    }

    
// 스위치 On/Off하여 녹음 모드인지 재생 모드인지 결정
    @IBAction func swRecordMode(_ sender: UISwitch) {

        if sender.isOn { // 녹음 모드일 경우 

            audioPlayer.stop()

            audioPlayer.currentTime = 0

            lblRecordTime!.text = convertNSTimeInterval2String(0)

            isRecordMode = true

            btnRecord.isEnabled = true

            lblRecordTime.isEnabled = true

        } else { // 재생 모드일 경우

            isRecordMode = false

            btnRecord.isEnabled = false

            lblRecordTime.isEnabled = false

            lblRecordTime.text = convertNSTimeInterval2String(0)

        }

        selectAudioFile()  // 모드에 따라 오디오 파일을 선택

        if !isRecordMode {  // 재생 모드일 때

            initplay()

        } else {  // 녹음 모드 일 때

            initRecord()

        }

    }

    

    @IBAction func btnRecord(_ sender: UIButton) {

        if (sender as AnyObject).titleLabel?.text == "Record" {
// Record일 때 녹음  중지
            audioRecorder.record()

            (sender as AnyObject).setTitle("Stop", for: UIControl.State())

            progressTimer = Timer.scheduledTimer(timeInterval: 0.1, target: self, selector: timeRecordSelector, userInfo: nil, repeats: true)

        } else { // stop일 때 녹음을 위한 초기화 

            audioRecorder.stop()

            progressTimer.invalidate()

            (sender as AnyObject).setTitle("Record", for: UIControl.State())

            btnPlay.isEnabled = true

            initplay()

        }

    }

    
// 0.1초마다 호출 및 녹음 시간 표시
    @objc func updateRecordTime() {

        lblRecordTime.text = convertNSTimeInterval2String(audioRecorder.currentTime)

    }

}
```
---
# 스크린샷

![image](https://user-images.githubusercontent.com/106370789/173602865-ababb528-ed1b-4c83-b3a3-f5075c89e430.png)

![image](https://user-images.githubusercontent.com/106370789/173602931-0852a584-a658-4535-87fb-9e3749912065.png)

![image](https://user-images.githubusercontent.com/106370789/173602994-85c3d9b3-f0db-4d72-aa11-f8c99a592668.png)

---
# 출처 
+ 스위프트로 아이폰 앱 만들기 입문 개정 6판

