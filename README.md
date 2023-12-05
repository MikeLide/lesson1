# lesson1
//
//  TextSelectorView.swift
//  CollageBuilder
//
//

import SwiftUI

struct TextSelectorView: View {
    
    @EnvironmentObject private var store: AppStore
    
    @State private var showFontSelector = false
    @State private var showTextEditor = false
    
    var body: some View {
        
        Section("Common settings") {
            commonSettings
        }
        
        if let text {
            textEditor
                .sheet(isPresented: $showTextEditor) {
                    TextEditorView(text: .init(
                        get: { text },
                        set: { dispatch(.changeText($0.text))}
                    ))
                    .presentationBackground(.thinMaterial)
                }
            
            Section("Font") {
                createFontEditor(for: text)
                .sheet(isPresented: $showFontSelector) {
                    FontSelectorView()
                }
            }
            
            Section("Paragraph") {
                createParagraphEditor(for: text)
            }
            
            Section("Appearance") {
                createAppearanceEditor(for: text)
            }
            
            Section("Animation") {
                animation
            }
            
            Section("manage") {
                remove
            }
            
            
        } else {
            EmptyView()
        }
    }
    
    private var animation: some View {
        AnimationSelectorView(animation: .init(
            get: { text?.animation },
            set: { dispatch(.changeAnimation($0)) }
        ))
    }
    
    private var commonSettings: some View {
        VStack {
            BlendModeSelectorView(blendMode: .init(
                get: { text?.blendMode ?? .normal },
                set: { dispatch(.changeBlendMode($0)) }
            ))
            ZPositionSelectorView(zPosition: .init(
                get: { text?.zPosition ?? 0 },
                set: { dispatch(.changeZPosition($0)) }
            ))
        }
    }
    
    private var remove: some View {
        HStack {
            Text("Remove")
            Spacer()
            Button {
                if let id = store.state.selectedElement?.textId {
                    store.dispatch(.changeCollage(
                        .removeText(id)
                    ))
                }
            } label: {
                Image(systemName: "trash.slash")
                    .font(.title2)
            }
        }
    }
    
    private var textEditor: some View {
        Button("Change Text") {
            showTextEditor.toggle()
        }
        .frame(maxWidth: .infinity)
    }
    
    private func createAppearanceEditor(for text: TextSettings) -> some View {
        VStack(spacing: 16) {
            ColorPicker(selection: .init(
                get: { Color(text.backgroundColor) },
                set: { dispatch(.changeBackgroundColor(UIColor($0)))}
            )) {
                Text("Background color")
            }
            CommonSliderView(
                value: .init(
                    get: { text.cornerRadius },
                    set: { dispatch(.changeCornerRadius($0)) }
                ),
                range: 0...50,
                title: "Corner radius"
            )
            .padding(.trailing, 5)
        }
    }
    
    private func createParagraphEditor(for text: TextSettings) -> some View {
        VStack(spacing: 16) {
            CommonSliderView(
                value: .init(
                    get: { text.kern },
                    set: { dispatch(.changeKern($0)) }
                ),
                range: 0...20,
                title: "Kent"
            )
            
            CommonSliderView(
                value: .init(
                    get: { text.lineSpacing },
                    set: { dispatch(.changeLineSpacing($0)) }
                ),
                range: 0...100,
                title: "Line spacing"
            )
            HStack {
                Text("Alignment")
                Spacer()
                Picker(
                    "",
                    selection: .init(
                        get: { text.alignment },
                        set: { dispatch(.changeAlignment($0)) }
                    )
                ) {
                    ForEach(TextSettings.TextAlignment.allCases, id: \.self) {
                        Text($0.rawValue)
                    }
                }
            }
        }
    }
    
    private func createFontEditor(for text: TextSettings) -> some View {
        VStack(spacing: 16) {
            CommonSliderView(
                value: .init(
                    get: { text.fontSize },
                    set: { dispatch(.changeSize($0)) }
                ),
                range: 10...100,
                title: "Size"
            )
            
            HStack {
                Text("Name")
                Spacer()
                Button {
                    showFontSelector.toggle()
                } label: {
                    Text(text.fontName)
                        .font(.custom(text.fontName, size: 14))
                }
            }
            
            ColorPicker(selection: .init(
                get: { Color(text.textColor) },
                set: { dispatch(.changeTextColor(UIColor($0)))}
            )) {
                Text("Text color")
            }
            .padding(.trailing, 5)
        }
    }
    
    private var text: TextSettings? {
        let text = store.state.collage.texts.first(where: {
            $0.id == store.state.selectedElement?.textId
        })
        
        return text
    }
    
    private func dispatch(_ action: TextModification) {
        guard let id = store.state.selectedElement?.textId else {
            return
        }
        
        store.dispatch(.changeCollage(.changeText(action, id: id)))
    }
}

struct TextSelectorView_Previews: PreviewProvider {
    static var previews: some View {
        List {
            TextSelectorView()
                .environmentObject(AppStore.preview)
        }
    }
}

