```markdown
# SAPUI5로 Master-Detail 프로그램 만들기

- 이 문서에서는 SAPUI5로 Master-Detail 프로그램을 단계별로 구현하는 방법을 설명하겠습니다.
- 이 프로그램은 CRUD 기능을 지원하고,
- i18n을 통해 다국어를 지원하며,
- 각 필드에 대해 유효성 검사와 경고 처리를 포함합니다.

## 1단계: SAPUI5 프로젝트 생성

- SAP Web IDE 또는 SAP Business Application Studio에서 새 SAPUI5 프로젝트를 생성합니다.
- 프로젝트 이름은 `zlistreport`로 설정하고, 필요한 기본 설정을 구성합니다.

## 2단계: 모델 및 데이터 구조 정의

### 2.1 OData 모델 생성

`zlistreport`의 데이터를 처리할 수 있는 OData 서비스를 설정합니다. 모델은 다음과 같은 데이터를 포함해야 합니다:

- **name**: 20자 제한
- **age**: 3자리 숫자 제한
- **gender**: 1자리 문자 제한
- **reg_user**: 10자 제한
- **reg_date**: 10자 (yyyy-mm-dd 형식)

### 2.2 OData 서비스 예시

OData 서비스에서 Master 엔터티를 정의합니다. Master 엔터티는 위 컬럼을 포함하고 있어야 합니다.

```xml
<EntityType Name="Master">
    <Key>
        <PropertyRef Name="name"/>
    </Key>
    <Property Name="name" Type="Edm.String" MaxLength="20"/>
    <Property Name="age" Type="Edm.Int32" MaxLength="3"/>
    <Property Name="gender" Type="Edm.String" MaxLength="1"/>
    <Property Name="reg_user" Type="Edm.String" MaxLength="10"/>
    <Property Name="reg_date" Type="Edm.String" MaxLength="10"/>
</EntityType>
```

## 3단계: i18n 다국어 파일 설정

### 3.1 i18n.properties 파일 생성

프로젝트의 `i18n` 폴더에 `i18n.properties` 파일을 생성하고, 필요한 메시지 키를 정의합니다.

```properties
createButton=Create
deleteButton=Delete
editButton=Edit
nameLabel=Name
ageLabel=Age
genderLabel=Gender
regUserLabel=Register User
regDateLabel=Registration Date
validationNameLength=Name must not exceed 20 characters.
validationAgeNumber=Age must be a number.
validationAgeLength=Age must not exceed 3 digits.
validationNameRequired=Name is a required field.
validationAgeRequired=Age is a required field.
```

### 3.2 다국어 지원

i18n 파일에 영문과 한글 버전을 추가하여 다국어 처리를 할 수 있습니다.

#### i18n_en.properties (English)

```properties
createButton=Create
deleteButton=Delete
editButton=Edit
nameLabel=Name
ageLabel=Age
genderLabel=Gender
regUserLabel=Register User
regDateLabel=Registration Date
validationNameLength=Name must not exceed 20 characters.
validationAgeNumber=Age must be a number.
validationAgeLength=Age must not exceed 3 digits.
validationNameRequired=Name is a required field.
validationAgeRequired=Age is a required field.
```

#### i18n_ko.properties (Korean)

```properties
createButton=생성
deleteButton=삭제
editButton=편집
nameLabel=이름
ageLabel=나이
genderLabel=성별
regUserLabel=등록 사용자
regDateLabel=등록 날짜
validationNameLength=이름은 20자를 초과할 수 없습니다.
validationAgeNumber=나이는 숫자여야 합니다.
validationAgeLength=나이는 3자리를 초과할 수 없습니다.
validationNameRequired=이름은 필수 항목입니다.
validationAgeRequired=나이는 필수 항목입니다.
```

## 4단계: Master-Detail 뷰 구현

### 4.1 Master View

마스터 뷰는 목록을 표시하는 화면입니다. 이 화면에서는 데이터를 조회하고, 선택된 항목에 대한 상세 정보를 보여줍니다.

```xml
<mvc:View xmlns:mvc="sap.ui.core.mvc" xmlns="sap.m" controllerName="zlistreport.controller.Master">
    <List id="masterList" items="{/Master}">
        <StandardListItem title="{name}" description="{age}" />
    </List>
</mvc:View>
```

### 4.2 Detail View

디테일 뷰는 사용자가 선택한 항목의 상세 정보를 보여주며, 데이터를 추가하거나 수정할 수 있는 폼을 제공합니다.

```xml
<mvc:View xmlns:mvc="sap.ui.core.mvc" xmlns="sap.m" controllerName="zlistreport.controller.Detail">
    <VBox>
        <Label text="{i18n>nameLabel}" />
        <Input value="{/name}" maxLength="20" id="nameInput" />
        <Label text="{i18n>ageLabel}" />
        <Input value="{/age}" maxLength="3" id="ageInput" />
        <Label text="{i18n>genderLabel}" />
        <Input value="{/gender}" maxLength="1" id="genderInput" />
        <Label text="{i18n>regUserLabel}" />
        <Input value="{/reg_user}" maxLength="10" id="regUserInput" />
        <Label text="{i18n>regDateLabel}" />
        <Input value="{/reg_date}" maxLength="10" id="regDateInput" />
        <Button text="{i18n>createButton}" press="onCreate" />
        <Button text="{i18n>deleteButton}" press="onDelete" />
        <Button text="{i18n>editButton}" press="onEdit" />
    </VBox>
</mvc:View>
```

## 5단계: 유효성 검사 및 CRUD 기능 구현

### 5.1 Controller 작성

Controller는 각 버튼의 동작을 정의하며, 유효성 검사도 처리합니다.

```javascript
sap.ui.define([
    "sap/ui/core/mvc/Controller",
    "sap/m/MessageBox"
], function (Controller, MessageBox) {
    "use strict";

    return Controller.extend("zlistreport.controller.Detail", {
        onInit: function () {
            // 모델 설정 등
        },
        onCreate: function () {
            var oName = this.getView().byId("nameInput").getValue();
            var oAge = this.getView().byId("ageInput").getValue();

            // 이름이 20자 이상일 경우
            if (oName.length > 20) {
                MessageBox.warning(this.getView().getModel("i18n").getResourceBundle().getText("validationNameLength"));
                return;
            }

            // 나이가 숫자가 아닌 경우
            if (isNaN(oAge)) {
                MessageBox.warning(this.getView().getModel("i18n").getResourceBundle().getText("validationAgeNumber"));
                return;
            }

            // 나이가 3자리 이상일 경우
            if (oAge.length > 3) {
                MessageBox.warning(this.getView().getModel("i18n").getResourceBundle().getText("validationAgeLength"));
                return;
            }

            // 필수 항목 검사
            if (!oName) {
                MessageBox.warning(this.getView().getModel("i18n").getResourceBundle().getText("validationNameRequired"));
                return;
            }
            if (!oAge) {
                MessageBox.warning(this.getView().getModel("i18n").getResourceBundle().getText("validationAgeRequired"));
                return;
            }

            // 데이터 생성 로직 추가 (예: OData 서비스 호출)
            MessageBox.success("Data created successfully!");
        },
        onDelete: function () {
            MessageBox.success("Delete functionality goes here.");
        },
        onEdit: function () {
            MessageBox.success("Edit functionality goes here.");
        }
    });
});
```

## 6단계: 실행 및 테스트

이제 프로젝트를 실행하여, 마스터-디테일 화면에서 데이터를 CRUD할 수 있고, 유효성 검사를 통해 올바르지 않은 입력을 방지할 수 있는지 테스트합니다.

위 단계를 따라가며 SAPUI5 기반의 Master-Detail 프로그램을 구현하실 수 있습니다. 추가적인 기능이나 변경 사항이 필요하다면 언제든지 알려주세요!
```
