# AlexSwift
//
//  TrimSelectorView.swift
//  CollageBuilder
//

import SwiftUI

struct TrimSelectorView: View {
    
    @Binding private var trim: VideoTrim?
    
    @State private var left: CGFloat
    @State private var right: CGFloat
    
    @State private var leftTranslation: CGFloat = 0
    @State private var rightTranslation: CGFloat = 0
    
    private var duration: CGFloat
    private var distance = 0.05
    private var width: CGFloat
    
    init(trim: Binding<VideoTrim?>,
         duration: CGFloat,
         width: CGFloat) {
        
        _trim = trim
        _left = State(
            initialValue: (trim.wrappedValue?.start ?? 0) / duration
        )
        _right = State(
            initialValue: (trim.wrappedValue?.end ?? duration) / duration
        )
        self.duration = duration
        self.width = width
    }
    
    var body: some View {
        ZStack {
            topElement
            leftElement
            rightElement
            bottomElement
        }
        .onChange(of: correctLeft) {
            setupTrim()
            trim?.start = $0 * duration
        }
        .onChange(of: correctRight) {
            setupTrim()
            trim?.end = $0 * duration
        }
    }
    
    private var bottomElement: some View {
        Rectangle()
            .fill(.green)
            .frame(width: width * (correctRight - correctLeft),
                   height: 3)
            .position(x: width * (correctRight + correctLeft) / 2, y: 40)
    }
    
    private var topElement: some View {
        Rectangle()
            .fill(.green)
            .frame(width: width * (correctRight - correctLeft),
                   height: 3)
            .position(x: width * (correctRight + correctLeft) / 2, y: 0)
    }
    
    private var rightElement: some View {
        Capsule()
            .fill(.green)
            .frame(width: 10, height: 50)
            .position(x: width * correctRight, y: 20)
            .gesture(DragGesture(minimumDistance: 0)
                .onChanged {
                    let translation = $0.translation.width / width
                    
                    if (translation + right) > 1 {
                        rightTranslation = 1 - right
                    } else if (translation + right - distance) < left {
                        rightTranslation = left + distance - right
                    } else {
                        rightTranslation = translation
                    }
                }
                .onEnded { _ in
                    right += rightTranslation
                    rightTranslation = 0
                }
            )
            .overlay {
                Text(createTime(for: Int(correctRight * duration)))
                    .font(.system(size: 12))
                    .position(x: width * correctRight, y: 60)
            }
    }
    
    private var leftElement: some View {
        Capsule()
            .fill(.green)
            .frame(width: 10, height: 50)
            .position(x: width * correctLeft, y: 20)
            .gesture(DragGesture(minimumDistance: 0)
                .onChanged {
                    let translation = $0.translation.width / width
                    
                    if (translation + left) < 0 {
                        leftTranslation = -left
                    } else if (translation + left + distance) > right {
                        leftTranslation = right - distance - left
                    } else {
                        leftTranslation = translation
                    }
                    
                }
                .onEnded { _ in
                    left += leftTranslation
                    leftTranslation = 0
                }
            )
            .overlay {
                Text(createTime(for: Int(correctLeft * duration)))
                    .font(.system(size: 12))
                    .position(x: width * correctLeft, y: 60)
            }
    }
    
    private func createTime(for seconds: Int) -> String {
        String(format: "%02i:%02i",
               seconds / 60,
               seconds % 60)
    }
    
    private func setupTrim() {
        guard trim == nil else { return }
        trim = .init(start: 0, end: duration)
    }
    
    private var correctRight: CGFloat {
        rightTranslation == 0
        ? right
        : right + rightTranslation
    }
    
    private var correctLeft: CGFloat {
        leftTranslation == 0
        ? left
        : left + leftTranslation
    }
    
}
