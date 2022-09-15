# swift_real_time_object_recognition

```swift
//
//  ViewController.swift
//  RealmTimeCameraObjectRecognition
//
//  Created by paige shin on 2022/09/16.
//

import UIKit
import AVKit
import Vision

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // MARK: - AVKIT
        let captureSession = AVCaptureSession()
        captureSession.sessionPreset = .photo
        
        // You can choose camera, audio. whatever you want
        // Video: Privacy - Camera Usage Description
        guard let captureDevice = AVCaptureDevice.default(for: .video) else { return }
        // Audio Input or Camera Input
        guard let input = try? AVCaptureDeviceInput(device: captureDevice) else { return }
        captureSession.addInput(input)
        captureSession.startRunning()
        
        // Preview layer
        let previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        view.layer.addSublayer(previewLayer)
        previewLayer.frame = view.frame
        
        // Add Output
        let dataOutput = AVCaptureVideoDataOutput()
        dataOutput
            .setSampleBufferDelegate(self, queue: DispatchQueue(label: "VideoQueue"))
        captureSession.addOutput(dataOutput)
    }


}


// MARK: AVKIT
extension ViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        // MARK: VISION
        guard let pixerBuffer: CVPixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        guard let model = try? VNCoreMLModel(for: SqueezeNet(configuration: MLModelConfiguration()).model) else { return }
        let request = VNCoreMLRequest(model: model) { finishedReq, error in
            guard let results = finishedReq.results as? [VNClassificationObservation] else { return }
            guard let firstObservation = results.first else { return }
            print(firstObservation.identifier)
            print(firstObservation.confidence)
        }
        try? VNImageRequestHandler(cvPixelBuffer: pixerBuffer, options: [:])
            .perform([request])
    }
    
}


```
